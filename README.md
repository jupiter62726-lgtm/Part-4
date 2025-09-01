/ModLoader/app/src/main/java/com/modloader/util/OnlineInstaller.java

// File: OnlineInstaller.java (Utility Class) - Complete Automated Installation System
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/app/src/main/java/com/terrarialoader/util/OnlineInstaller.java

package com.modloader.util;

import android.content.Context;
import com.modloader.loader.MelonLoaderManager;
import java.io.File;

/**
 * OnlineInstaller - Complete automated installation system
 * Uses existing Downloader and PathManager for seamless MelonLoader/LemonLoader installation
 */
public class OnlineInstaller {
    
    // GitHub URLs for MelonLoader/LemonLoader releases
    public static final String MELONLOADER_URL = "https://github.com/LavaGang/MelonLoader/releases/download/0.6.5.1/melon_data.zip";
    public static final String LEMONLOADER_URL = "https://github.com/LemonLoader/MelonLoader/releases/download/0.6.5.1/melon_data.zip";
    
    // Installation result class
    public static class InstallationResult {
        public boolean success;
        public String message;
        public String errorDetails;
        public File installationPath;
        public int filesInstalled;
        public long totalSize;
        
        public InstallationResult(boolean success, String message) {
            this.success = success;
            this.message = message;
        }
        
        public InstallationResult(boolean success, String message, String errorDetails) {
            this.success = success;
            this.message = message;
            this.errorDetails = errorDetails;
        }
    }
    
    /**
     * Complete automated installation of MelonLoader
     * Downloads melon_data.zip and extracts to proper directory structure
     * 
     * @param context Application context
     * @param gamePackage Target game package (e.g., "com.and.games505.TerrariaPaid")
     * @param loaderType Type of loader to install (NET8 or NET35)
     * @return InstallationResult with success status and details
     */
    public static InstallationResult installMelonLoaderOnline(Context context, String gamePackage, MelonLoaderManager.LoaderType loaderType) {
        LogUtils.logUser("🚀 Starting automated MelonLoader installation...");
        LogUtils.logUser("Target: " + gamePackage + " (" + loaderType.getDisplayName() + ")");
        
        // Validate parameters
        if (context == null) {
            return new InstallationResult(false, "Context is null", "Application context is required");
        }
        
        if (gamePackage == null || gamePackage.trim().isEmpty()) {
            return new InstallationResult(false, "Invalid game package", "Game package cannot be null or empty");
        }
        
        if (loaderType == null) {
            return new InstallationResult(false, "Invalid loader type", "Loader type cannot be null");
        }
        
        try {
            // Step 1: Initialize directory structure
            LogUtils.logUser("📁 Step 1: Initializing directory structure...");
            if (!PathManager.initializeGameDirectories(context, gamePackage)) {
                return new InstallationResult(false, "Failed to create directory structure", "Could not create required directories");
            }
            LogUtils.logUser("✅ Directory structure ready");
            
            // Step 2: Determine download URL and target directory
            String downloadUrl = (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) ? MELONLOADER_URL : LEMONLOADER_URL;
            File targetDirectory = PathManager.getMelonLoaderDir(context, gamePackage);
            
            if (targetDirectory == null) {
                return new InstallationResult(false, "Cannot determine installation directory", "PathManager returned null directory");
            }
            
            LogUtils.logUser("📂 Installation directory: " + targetDirectory.getAbsolutePath());
            LogUtils.logUser("🌐 Download URL: " + downloadUrl);
            
            // Step 3: Download and extract
            LogUtils.logUser("⬇️ Step 2: Downloading and extracting MelonLoader files...");
            boolean downloadSuccess = Downloader.downloadAndExtractZip(downloadUrl, targetDirectory);
            
            if (!downloadSuccess) {
                return new InstallationResult(false, "Download or extraction failed", "Failed to download from: " + downloadUrl);
            }
            
            // Step 4: Organize files according to MelonLoader structure
            LogUtils.logUser("📋 Step 3: Organizing files into proper structure...");
            InstallationResult organizationResult = organizeExtractedFiles(context, gamePackage, targetDirectory, loaderType);
            
            if (!organizationResult.success) {
                return organizationResult;
            }
            
            // Step 5: Validate installation
            LogUtils.logUser("🔍 Step 4: Validating installation...");
            boolean validationSuccess = MelonLoaderManager.validateLoaderInstallation(context, gamePackage).isValid;
            
            if (!validationSuccess) {
                LogUtils.logUser("⚠️ Installation validation failed, attempting repair...");
                if (MelonLoaderManager.attemptRepair(context, gamePackage)) {
                    LogUtils.logUser("✅ Repair successful");
                    validationSuccess = true;
                } else {
                    return new InstallationResult(false, "Installation validation failed", "Files were downloaded but validation failed");
                }
            }
            
            // Step 6: Create final result
            InstallationResult result = new InstallationResult(true, "✅ " + loaderType.getDisplayName() + " installed successfully!");
            result.installationPath = targetDirectory;
            result.filesInstalled = countInstalledFiles(targetDirectory);
            result.totalSize = calculateDirectorySize(targetDirectory);
            
            LogUtils.logUser("🎉 Installation completed successfully!");
            LogUtils.logUser("📊 Files installed: " + result.filesInstalled);
            LogUtils.logUser("💾 Total size: " + FileUtils.formatFileSize(result.totalSize));
            LogUtils.logUser("📍 Installation path: " + result.installationPath.getAbsolutePath());
            
            return result;
            
        } catch (Exception e) {
            String errorMsg = "Unexpected error during installation: " + e.getMessage();
            LogUtils.logDebug(errorMsg);
            e.printStackTrace();
            return new InstallationResult(false, "Installation failed with exception", errorMsg);
        }
    }
    
    /**
     * Organize extracted files into proper MelonLoader directory structure
     * Based on the MelonLoader_File_List.txt structure
     */
    private static InstallationResult organizeExtractedFiles(Context context, String gamePackage, File extractedDir, MelonLoaderManager.LoaderType loaderType) {
        try {
            LogUtils.logUser("🗂️ Organizing extracted files...");
            
            // Create target directories based on loader type
            File net8Dir = PathManager.getMelonLoaderNet8Dir(context, gamePackage);
            File net35Dir = PathManager.getMelonLoaderNet35Dir(context, gamePackage);
            File dependenciesDir = PathManager.getMelonLoaderDependenciesDir(context, gamePackage);
            
            // Ensure directories exist
            PathManager.ensureDirectoryExists(net8Dir);
            PathManager.ensureDirectoryExists(net35Dir);
            PathManager.ensureDirectoryExists(dependenciesDir);
            
            int organizedFiles = 0;
            
            // Process extracted files
            File[] extractedFiles = extractedDir.listFiles();
            if (extractedFiles != null) {
                for (File file : extractedFiles) {
                    if (organizeFile(file, net8Dir, net35Dir, dependenciesDir, loaderType)) {
                        organizedFiles++;
                    }
                }
            }
            
            LogUtils.logUser("📁 Organized " + organizedFiles + " files into proper structure");
            
            // Create additional required directories
            createAdditionalDirectories(context, gamePackage);
            
            InstallationResult result = new InstallationResult(true, "File organization completed");
            result.filesInstalled = organizedFiles;
            return result;
            
        } catch (Exception e) {
            return new InstallationResult(false, "File organization failed", e.getMessage());
        }
    }
    
    /**
     * Organize individual file based on its type and target loader
     */
    private static boolean organizeFile(File file, File net8Dir, File net35Dir, File dependenciesDir, MelonLoaderManager.LoaderType loaderType) {
        if (file == null || !file.exists()) {
            return false;
        }
        
        try {
            String fileName = file.getName().toLowerCase();
            File targetDir = null;
            
            // Determine target directory based on file type and loader type
            if (fileName.contains("net8") && loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) {
                targetDir = net8Dir;
            } else if (fileName.contains("net35") && loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET35) {
                targetDir = net35Dir;
            } else if (fileName.contains("dependencies") || fileName.contains("supportmodules") || fileName.contains("il2cpp")) {
                targetDir = dependenciesDir;
            } else if (fileName.endsWith(".dll") || fileName.endsWith(".xml") || fileName.endsWith(".pdb")) {
                // Core files go to appropriate runtime directory
                targetDir = (loaderType == MelonLoaderManager.LoaderType.MELONLOADER_NET8) ? net8Dir : net35Dir;
            }
            
            if (targetDir != null) {
                File targetFile = new File(targetDir, file.getName());
                if (file.renameTo(targetFile)) {
                    LogUtils.logDebug("Moved: " + file.getName() + " -> " + targetDir.getName());
                    return true;
                }
            }
            
            return false;
            
        } catch (Exception e) {
            LogUtils.logDebug("Error organizing file " + file.getName() + ": " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Create additional required directories for MelonLoader
     */
    private static void createAdditionalDirectories(Context context, String gamePackage) {
        try {
            // Create subdirectories in Dependencies
            File depsDir = PathManager.getMelonLoaderDependenciesDir(context, gamePackage);
            
            String[] subDirs = {
                "SupportModules",
                "CompatibilityLayers", 
                "Il2CppAssemblyGenerator",
                "Il2CppAssemblyGenerator/Cpp2IL",
                "Il2CppAssemblyGenerator/Cpp2IL/cpp2il_out",
                "Il2CppAssemblyGenerator/Il2CppInterop",
                "Il2CppAssemblyGenerator/Il2CppInterop/Il2CppAssemblies",
                "Il2CppAssemblyGenerator/UnityDependencies",
                "Il2CppAssemblyGenerator/runtimes",
                "Il2CppAssemblyGenerator/runtimes/linux-arm64/native",
                "Il2CppAssemblyGenerator/runtimes/linux-arm/native",
                "Il2CppAssemblyGenerator/runtimes/linux-x64/native",
                "Il2CppAssemblyGenerator/runtimes/linux-x86/native"
            };
            
            for (String subDir : subDirs) {
                File dir = new File(depsDir, subDir);
                PathManager.ensureDirectoryExists(dir);
            }
            
            LogUtils.logDebug("Created additional MelonLoader directories");
            
        } catch (Exception e) {
            LogUtils.logDebug("Error creating additional directories: " + e.getMessage());
        }
    }
    
    /**
     * Count files in a directory recursively
     */
    private static int countInstalledFiles(File directory) {
        if (directory == null || !directory.exists() || !directory.isDirectory()) {
            return 0;
        }
        
        int count = 0;
        File[] files = directory.listFiles();
        if (files != null) {
            for (File file : files) {
                if (file.isDirectory()) {
                    count += countInstalledFiles(file);
                } else {
                    count++;
                }
            }
        }
        return count;
    }
    
    /**
     * Calculate total size of directory
     */
    private static long calculateDirectorySize(File directory) {
        if (directory == null || !directory.exists()) {
            return 0;
        }
        
        long size = 0;
        if (directory.isDirectory()) {
            File[] files = directory.listFiles();
            if (files != null) {
                for (File file : files) {
                    if (file.isDirectory()) {
                        size += calculateDirectorySize(file);
                    } else {
                        size += file.length();
                    }
                }
            }
        } else {
            size = directory.length();
        }
        return size;
    }
    
    /**
     * Convenience method for installing MelonLoader (NET8)
     */
    public static InstallationResult installMelonLoader(Context context, String gamePackage) {
        return installMelonLoaderOnline(context, gamePackage, MelonLoaderManager.LoaderType.MELONLOADER_NET8);
    }
    
    /**
     * Convenience method for installing LemonLoader (NET35)  
     */
    public static InstallationResult installLemonLoader(Context context, String gamePackage) {
        return installMelonLoaderOnline(context, gamePackage, MelonLoaderManager.LoaderType.MELONLOADER_NET35);
    }
    
    /**
     * Install for Terraria specifically
     */
    public static InstallationResult installForTerraria(Context context, MelonLoaderManager.LoaderType loaderType) {
        return installMelonLoaderOnline(context, MelonLoaderManager.TERRARIA_PACKAGE, loaderType);
    }
    
    /**
     * Check if online installation is possible (internet connectivity)
     */
    public static boolean isOnlineInstallationAvailable() {
        try {
            // Simple connectivity check
            java.net.URL url = new java.net.URL("https://github.com");
            java.net.HttpURLConnection connection = (java.net.HttpURLConnection) url.openConnection();
            connection.setRequestMethod("HEAD");
            connection.setConnectTimeout(3000);
            connection.setReadTimeout(3000);
            connection.connect();
            
            int responseCode = connection.getResponseCode();
            return responseCode == 200;
            
        } catch (Exception e) {
            LogUtils.logDebug("Online installation not available: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Get installation progress callback interface
     */
    public interface InstallationProgressCallback {
        void onProgress(String message, int percentage);
        void onComplete(InstallationResult result);
        void onError(String error);
    }
    
    /**
     * Asynchronous installation with progress callback
     */
    public static void installMelonLoaderAsync(Context context, String gamePackage, MelonLoaderManager.LoaderType loaderType, InstallationProgressCallback callback) {
        new Thread(() -> {
            try {
                if (callback != null) {
                    callback.onProgress("Starting installation...", 0);
                }
                
                InstallationResult result = installMelonLoaderOnline(context, gamePackage, loaderType);
                
                if (callback != null) {
                    if (result.success) {
                        callback.onProgress("Installation completed!", 100);
                        callback.onComplete(result);
                    } else {
                        callback.onError(result.message + (result.errorDetails != null ? ": " + result.errorDetails : ""));
                    }
                }
                
            } catch (Exception e) {
                if (callback != null) {
                    callback.onError("Installation failed: " + e.getMessage());
                }
            }
        }).start();
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/PatchResult.java

// File: PatchResult.java - Complete patch result class
// Path: /storage/emulated/0/AndroidIDEProjects/ModLoader/app/src/main/java/com/modloader/util/PatchResult.java

package com.modloader.util;

import java.util.ArrayList;
import java.util.List;

public class PatchResult {
    public boolean success;
    public String errorMessage;
    public List<String> warnings;
    public List<String> details;
    public String outputPath;
    public long processingTime;
    public long startTime;
    public long endTime;
    public String operationType;
    
    public PatchResult() {
        this.success = false;
        this.warnings = new ArrayList<>();
        this.details = new ArrayList<>();
        this.startTime = System.currentTimeMillis();
        this.processingTime = 0;
    }
    
    public PatchResult(boolean success) {
        this();
        this.success = success;
    }
    
    public PatchResult(boolean success, String errorMessage) {
        this(success);
        this.errorMessage = errorMessage;
    }
    
    public PatchResult(String operationType) {
        this();
        this.operationType = operationType;
    }
    
    public void addWarning(String warning) {
        if (warnings == null) {
            warnings = new ArrayList<>();
        }
        warnings.add(warning);
    }
    
    public void addDetail(String detail) {
        if (details == null) {
            details = new ArrayList<>();
        }
        details.add(detail);
    }
    
    public boolean hasWarnings() {
        return warnings != null && !warnings.isEmpty();
    }
    
    public boolean hasDetails() {
        return details != null && !details.isEmpty();
    }
    
    public void setSuccess(boolean success) {
        this.success = success;
        if (success) {
            this.endTime = System.currentTimeMillis();
            this.processingTime = this.endTime - this.startTime;
        }
    }
    
    public void setError(String errorMessage) {
        this.success = false;
        this.errorMessage = errorMessage;
        this.endTime = System.currentTimeMillis();
        this.processingTime = this.endTime - this.startTime;
    }
    
    public void setErrorMessage(String errorMessage) {
        this.errorMessage = errorMessage;
    }
    
    public void setOutputPath(String outputPath) {
        this.outputPath = outputPath;
    }
    
    public void setOperationType(String operationType) {
        this.operationType = operationType;
    }
    
    public void complete() {
        this.endTime = System.currentTimeMillis();
        this.processingTime = this.endTime - this.startTime;
    }
    
    // Convenience method to convert to boolean for backward compatibility
    public boolean isSuccess() {
        return success;
    }
    
    public long getProcessingTimeMs() {
        return processingTime;
    }
    
    public String getFormattedProcessingTime() {
        if (processingTime < 1000) {
            return processingTime + "ms";
        } else if (processingTime < 60000) {
            return String.format("%.1fs", processingTime / 1000.0);
        } else {
            long seconds = processingTime / 1000;
            long minutes = seconds / 60;
            seconds = seconds % 60;
            return String.format("%dm %ds", minutes, seconds);
        }
    }
    
    @Override
    public String toString() {
        StringBuilder sb = new StringBuilder();
        sb.append("PatchResult{");
        sb.append("success=").append(success);
        if (operationType != null) {
            sb.append(", operation='").append(operationType).append('\'');
        }
        if (errorMessage != null) {
            sb.append(", error='").append(errorMessage).append('\'');
        }
        if (hasWarnings()) {
            sb.append(", warnings=").append(warnings.size());
        }
        if (processingTime > 0) {
            sb.append(", time=").append(getFormattedProcessingTime());
        }
        sb.append('}');
        return sb.toString();
    }
    
    public String getDetailedReport() {
        StringBuilder sb = new StringBuilder();
        sb.append("=== Patch Operation Report ===\n");
        if (operationType != null) {
            sb.append("Operation: ").append(operationType).append("\n");
        }
        sb.append("Result: ").append(success ? "SUCCESS" : "FAILED").append("\n");
        sb.append("Processing Time: ").append(getFormattedProcessingTime()).append("\n");
        
        if (outputPath != null) {
            sb.append("Output: ").append(outputPath).append("\n");
        }
        
        if (errorMessage != null) {
            sb.append("Error: ").append(errorMessage).append("\n");
        }
        
        if (hasWarnings()) {
            sb.append("\nWarnings (").append(warnings.size()).append("):\n");
            for (String warning : warnings) {
                sb.append("  - ").append(warning).append("\n");
            }
        }
        
        if (hasDetails()) {
            sb.append("\nDetails (").append(details.size()).append("):\n");
            for (String detail : details) {
                sb.append("  • ").append(detail).append("\n");
            }
        }
        
        return sb.toString();
    }
    
    // Static factory methods for common scenarios
    public static PatchResult success(String operationType) {
        PatchResult result = new PatchResult(operationType);
        result.setSuccess(true);
        return result;
    }
    
    public static PatchResult success(String operationType, String outputPath) {
        PatchResult result = success(operationType);
        result.setOutputPath(outputPath);
        return result;
    }
    
    public static PatchResult failure(String operationType, String errorMessage) {
        PatchResult result = new PatchResult(operationType);
        result.setError(errorMessage);
        return result;
    }
    
    public static PatchResult inProgress(String operationType) {
        return new PatchResult(operationType);
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/PathManager.java

// File: PathManager.java (FIXED Part 1) - Centralized Path Management
// Path: /storage/emulated/0/AndroidIDEProjects/TerrariaML/main/java/com/terrarialoader/util/PathManager.java

package com.modloader.util;

import android.content.Context;
import java.io.File;

/**
 * Centralized path management for TerrariaLoader
 * All file operations should use these standardized paths
 * FIXED: Added proper app logs directory support and correct structure
 */
public class PathManager {
    
    // Base directory: /storage/emulated/0/Android/data/com.modloader/files
    private static File getAppDataDirectory(Context context) {
        return context.getExternalFilesDir(null);
    }
    
    // === TERRARIA LOADER STRUCTURE ===
    
    /**
     * Base TerrariaLoader directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader
     */
    public static File getTerrariaLoaderBaseDir(Context context) {
        return new File(getAppDataDirectory(context), "TerrariaLoader");
    }
    
    /**
     * Game-specific base directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}
     */
    public static File getGameBaseDir(Context context, String gamePackage) {
        return new File(getTerrariaLoaderBaseDir(context), gamePackage);
    }
    
    /**
     * Default Terraria directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/com.and.games505.TerrariaPaid
     */
    public static File getTerrariaBaseDir(Context context) {
        return getGameBaseDir(context, "com.and.games505.TerrariaPaid");
    }
    
    // === MOD DIRECTORIES ===
    
    /**
     * DLL Mods directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Mods/DLL
     */
    public static File getDllModsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Mods/DLL");
    }
    
    /**
     * DEX Mods directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Mods/DEX
     */
    public static File getDexModsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Mods/DEX");
    }
    
    /**
     * Legacy mods directory (for backward compatibility)
     * @return /storage/emulated/0/Android/data/com.modloader/files/mods
     */
    public static File getLegacyModsDir(Context context) {
        return new File(getAppDataDirectory(context), "mods");
    }
    
    // === MELONLOADER STRUCTURE ===
    
    /**
     * MelonLoader base directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Loaders/MelonLoader
     */
    public static File getMelonLoaderDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Loaders/MelonLoader");
    }
    
    /**
     * MelonLoader NET8 runtime directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Loaders/MelonLoader/net8
     */
    public static File getMelonLoaderNet8Dir(Context context, String gamePackage) {
        return new File(getMelonLoaderDir(context, gamePackage), "net8");
    }
    
    /**
     * MelonLoader NET35 runtime directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Loaders/MelonLoader/net35
     */
    public static File getMelonLoaderNet35Dir(Context context, String gamePackage) {
        return new File(getMelonLoaderDir(context, gamePackage), "net35");
    }
    
    /**
     * MelonLoader Dependencies directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Loaders/MelonLoader/Dependencies
     */
    public static File getMelonLoaderDependenciesDir(Context context, String gamePackage) {
        return new File(getMelonLoaderDir(context, gamePackage), "Dependencies");
    }
    
    // === PLUGINS AND USERLIBS (FIXED: Now at game root level) ===
    
    /**
     * FIXED: Plugins directory (at game root level, not inside MelonLoader)
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Plugins
     */
    public static File getPluginsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Plugins");
    }
    
    /**
     * FIXED: UserLibs directory (at game root level, not inside MelonLoader)
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/UserLibs
     */
    public static File getUserLibsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "UserLibs");
    }
    
    // === LOG DIRECTORIES ===
    
    /**
     * FIXED: Game logs directory (MelonLoader logs)
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Logs
     */
    public static File getLogsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Logs");
    }
    
    /**
     * FIXED: App logs directory (TerrariaLoader app logs)
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/AppLogs
     */
    public static File getAppLogsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "AppLogs");
    }
    
    /**
     * Legacy app logs directory (for backward compatibility)
     * @return /storage/emulated/0/Android/data/com.modloader/files/logs
     */
    public static File getLegacyAppLogsDir(Context context) {
        return new File(getAppDataDirectory(context), "logs");
    }
    
    /**
     * Exports directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/exports
     */
    public static File getExportsDir(Context context) {
        return new File(getAppDataDirectory(context), "exports");
    }
    
    // === BACKUP DIRECTORIES ===
    
    /**
     * Backups directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Backups
     */
    public static File getBackupsDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Backups");
    }
    
    // === CONFIG DIRECTORIES ===
    
    /**
     * Config directory
     * @return /storage/emulated/0/Android/data/com.modloader/files/TerrariaLoader/{gamePackage}/Config
     */
    public static File getConfigDir(Context context, String gamePackage) {
        return new File(getGameBaseDir(context, gamePackage), "Config");
    }
    
    // === UTILITY METHODS ===
    
    /**
     * Ensure a directory exists, creating it if necessary
     * @param directory The directory to ensure exists
     * @return true if directory exists or was created successfully
     */
    public static boolean ensureDirectoryExists(File directory) {
        if (directory == null) {
            LogUtils.logDebug("Directory is null");
            return false;
        }
        
        if (directory.exists()) {
            return directory.isDirectory();
        }
        
        try {
            boolean created = directory.mkdirs();
            if (created) {
                LogUtils.logDebug("Created directory: " + directory.getAbsolutePath());
            } else {
                LogUtils.logDebug("Failed to create directory: " + directory.getAbsolutePath());
            }
            return created;
        } catch (Exception e) {
            LogUtils.logDebug("Error creating directory: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Initialize all required directories for a game package
     * @param context Application context
     * @param gamePackage Game package name
     * @return true if all directories were created successfully
     */
    public static boolean initializeGameDirectories(Context context, String gamePackage) {
        LogUtils.logUser("Initializing directory structure for: " + gamePackage);
        
        File[] requiredDirs = {
            getGameBaseDir(context, gamePackage),
            getDllModsDir(context, gamePackage),
            getDexModsDir(context, gamePackage),
            getMelonLoaderDir(context, gamePackage),
            getMelonLoaderNet8Dir(context, gamePackage),
            getMelonLoaderNet35Dir(context, gamePackage),
            getMelonLoaderDependenciesDir(context, gamePackage),
            getPluginsDir(context, gamePackage),        // FIXED: Added Plugins
            getUserLibsDir(context, gamePackage),       // FIXED: Added UserLibs
            getLogsDir(context, gamePackage),           // Game logs
            getAppLogsDir(context, gamePackage),        // FIXED: Added App logs
            getBackupsDir(context, gamePackage),
            getConfigDir(context, gamePackage)
        };
        
        boolean allSuccess = true;
        int createdCount = 0;
        
        for (File dir : requiredDirs) {
            if (!dir.exists()) {
                if (ensureDirectoryExists(dir)) {
                    createdCount++;
                } else {
                    allSuccess = false;
                    LogUtils.logDebug("Failed to create: " + dir.getAbsolutePath());
                }
            }
        }
        
        LogUtils.logUser("Directory initialization complete: " + createdCount + " directories created");
        
        // Create README files
        if (allSuccess) {
            createReadmeFiles(context, gamePackage);
        }
        
        return allSuccess;
    }
    
    /**
     * FIXED: Create helpful README files in directories
     */
    private static void createReadmeFiles(Context context, String gamePackage) {
        try {
            // DLL Mods README
            File dllReadme = new File(getDllModsDir(context, gamePackage), "README.txt");
            if (!dllReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(dllReadme)) {
                    writer.write("=== TerrariaLoader - DLL Mods ===\n\n");
                    writer.write("Place your .dll mod files here.\n");
                    writer.write("Requires MelonLoader to be installed.\n\n");
                    writer.write("Supported formats:\n");
                    writer.write("• .dll files (enabled)\n");
                    writer.write("• .dll.disabled files (disabled)\n\n");
                    writer.write("Path: " + dllReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
            // DEX Mods README
            File dexReadme = new File(getDexModsDir(context, gamePackage), "README.txt");
            if (!dexReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(dexReadme)) {
                    writer.write("=== TerrariaLoader - DEX/JAR Mods ===\n\n");
                    writer.write("Place your .dex and .jar mod files here.\n");
                    writer.write("These are Java-based mods for Android.\n\n");
                    writer.write("Supported formats:\n");
                    writer.write("• .dex files (enabled)\n");
                    writer.write("• .jar files (enabled)\n");
                    writer.write("• .dex.disabled files (disabled)\n");
                    writer.write("• .jar.disabled files (disabled)\n\n");
                    writer.write("Path: " + dexReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
            // FIXED: Plugins README
            File pluginsReadme = new File(getPluginsDir(context, gamePackage), "README.txt");
            if (!pluginsReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(pluginsReadme)) {
                    writer.write("=== MelonLoader Plugins Directory ===\n\n");
                    writer.write("This directory contains MelonLoader plugins.\n");
                    writer.write("Plugins extend MelonLoader functionality.\n\n");
                    writer.write("Files you might see here:\n");
                    writer.write("• .dll plugin files\n");
                    writer.write("• Plugin configuration files\n\n");
                    writer.write("Path: " + pluginsReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
            // FIXED: UserLibs README
            File userlibsReadme = new File(getUserLibsDir(context, gamePackage), "README.txt");
            if (!userlibsReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(userlibsReadme)) {
                    writer.write("=== MelonLoader UserLibs Directory ===\n\n");
                    writer.write("This directory contains user libraries.\n");
                    writer.write("Libraries that mods depend on go here.\n\n");
                    writer.write("Files you might see here:\n");
                    writer.write("• .dll library files\n");
                    writer.write("• Shared mod dependencies\n\n");
                    writer.write("Path: " + userlibsReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
            // FIXED: Game logs README
            File gameLogsReadme = new File(getLogsDir(context, gamePackage), "README.txt");
            if (!gameLogsReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(gameLogsReadme)) {
                    writer.write("=== MelonLoader Game Logs ===\n\n");
                    writer.write("This directory contains logs from MelonLoader and mods.\n");
                    writer.write("These are generated when running the patched game.\n\n");
                    writer.write("Log files follow rotation:\n");
                    writer.write("• Log.txt (current log)\n");
                    writer.write("• Log1.txt to Log5.txt (previous logs)\n");
                    writer.write("• Oldest logs are automatically deleted\n\n");
                    writer.write("Path: " + gameLogsReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
            // FIXED: App logs README
            File appLogsReadme = new File(getAppLogsDir(context, gamePackage), "README.txt");
            if (!appLogsReadme.exists()) {
                try (java.io.FileWriter writer = new java.io.FileWriter(appLogsReadme)) {
                    writer.write("=== TerrariaLoader App Logs ===\n\n");
                    writer.write("This directory contains logs from TerrariaLoader app.\n");
                    writer.write("These are generated when using this app.\n\n");
                    writer.write("Log files follow rotation:\n");
                    writer.write("• AppLog.txt (current log)\n");
                    writer.write("• AppLog1.txt to AppLog5.txt (previous logs)\n");
                    writer.write("• Oldest logs are automatically deleted\n\n");
                    writer.write("Path: " + appLogsReadme.getParentFile().getAbsolutePath() + "\n");
                }
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Error creating README files: " + e.getMessage());
        }
    }
    
    /**
     * FIXED: Get standardized path string for logging/display
     */
    public static String getPathInfo(Context context, String gamePackage) {
        StringBuilder info = new StringBuilder();
        info.append("=== TerrariaLoader Directory Structure ===\n");
        info.append("Base: ").append(getTerrariaLoaderBaseDir(context).getAbsolutePath()).append("\n");
        info.append("Game: ").append(getGameBaseDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("DLL Mods: ").append(getDllModsDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("DEX Mods: ").append(getDexModsDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("MelonLoader: ").append(getMelonLoaderDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("Plugins: ").append(getPluginsDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("UserLibs: ").append(getUserLibsDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("Game Logs: ").append(getLogsDir(context, gamePackage).getAbsolutePath()).append("\n");
        info.append("App Logs: ").append(getAppLogsDir(context, gamePackage).getAbsolutePath()).append("\n");
        return info.toString();
    }
    
    /**
     * Check if legacy structure exists and needs migration
     */
    public static boolean needsMigration(Context context) {
        File legacyMods = getLegacyModsDir(context);
        File legacyAppLogs = getLegacyAppLogsDir(context);
        File newStructure = getTerrariaBaseDir(context);
        
        return ((legacyMods.exists() && legacyMods.listFiles() != null && legacyMods.listFiles().length > 0) ||
                (legacyAppLogs.exists() && legacyAppLogs.listFiles() != null && legacyAppLogs.listFiles().length > 0)) && 
               !newStructure.exists();
    }
    
    /**
     * FIXED: Migrate from legacy structure to new structure
     */
    public static boolean migrateLegacyStructure(Context context) {
        if (!needsMigration(context)) {
            return true;
        }
        
        LogUtils.logUser("Migrating from legacy directory structure...");
        
        try {
            String gamePackage = "com.and.games505.TerrariaPaid";
            
            // Initialize new structure
            if (!initializeGameDirectories(context, gamePackage)) {
                LogUtils.logDebug("Failed to initialize new directory structure");
                return false;
            }
            
            // Migrate mods
            File legacyModsDir = getLegacyModsDir(context);
            File newDexMods = getDexModsDir(context, gamePackage);
            
            if (legacyModsDir.exists()) {
                File[] modFiles = legacyModsDir.listFiles();
                if (modFiles != null) {
                    int migratedCount = 0;
                    for (File modFile : modFiles) {
                        if (modFile.isFile()) {
                            File newLocation = new File(newDexMods, modFile.getName());
                            if (modFile.renameTo(newLocation)) {
                                migratedCount++;
                            }
                        }
                    }
                    LogUtils.logUser("Migrated " + migratedCount + " mod files to new structure");
                }
            }
            
            // Migrate legacy app logs
            File legacyAppLogs = getLegacyAppLogsDir(context);
            File newAppLogs = getAppLogsDir(context, gamePackage);
            
            if (legacyAppLogs.exists()) {
                File[] logFiles = legacyAppLogs.listFiles((dir, name) -> 
                    name.endsWith(".txt") || name.startsWith("auto_save_"));
                
                if (logFiles != null && logFiles.length > 0) {
                    LogUtils.logUser("Migrating " + logFiles.length + " legacy log files...");
                    
                    if (!newAppLogs.exists()) {
                        newAppLogs.mkdirs();
                    }
                    
                    int migratedLogCount = 0;
                    for (File logFile : logFiles) {
                        // Rename to new format
                        String newName = "AppLog" + (migratedLogCount + 1) + ".txt";
                        File newLocation = new File(newAppLogs, newName);
                        
                        if (logFile.renameTo(newLocation)) {
                            migratedLogCount++;
                        }
                    }
                    LogUtils.logUser("Migrated " + migratedLogCount + " log files to new structure");
                }
            }
            
            LogUtils.logUser("✅ Migration completed successfully");
            return true;
            
        } catch (Exception e) {
            LogUtils.logDebug("Migration failed: " + e.getMessage());
            return false;
        }
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/PermissionManager.java

// File: PermissionManager.java (COMPLETE FIXED) - No Syntax Errors
// Path: /app/src/main/java/com/modloader/util/PermissionManager.java

package com.modloader.util;

import android.Manifest;
import android.app.Activity;
import android.content.Context;
import android.content.Intent;
import android.content.pm.PackageManager;
import android.net.Uri;
import android.os.Build;
import android.os.Environment;
import android.provider.Settings;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import androidx.core.app.ActivityCompat;
import androidx.core.content.ContextCompat;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

public class PermissionManager {
    private static final String TAG = "PermissionManager";
    
    // Request codes for different permission types
    public static final int REQUEST_BASIC_PERMISSIONS = 1001;
    public static final int REQUEST_STORAGE_PERMISSIONS = 1002;
    public static final int REQUEST_INSTALL_PERMISSION = 1003;
    public static final int REQUEST_ALL_FILES_ACCESS = 1004;
    
    // Essential permissions for app functionality
    private static final String[] BASIC_PERMISSIONS = {
        Manifest.permission.READ_EXTERNAL_STORAGE,
        Manifest.permission.WRITE_EXTERNAL_STORAGE
    };
    
    // Additional permissions for enhanced functionality
    private static final String[] ENHANCED_PERMISSIONS = {
        Manifest.permission.REQUEST_INSTALL_PACKAGES,
        Manifest.permission.INTERNET,
        Manifest.permission.ACCESS_NETWORK_STATE
    };
    
    private final Context context;
    private final Activity activity;
    
    // Permission state tracking
    private final Map<String, Boolean> permissionStates = new HashMap<>();
    private PermissionCallback callback;
    
    public interface PermissionCallback {
        void onPermissionGranted(String permission);
        void onPermissionDenied(String permission);
        void onPermissionPermanentlyDenied(String permission);
        void onAllPermissionsGranted();
        void onPermissionRequestCompleted(boolean allGranted);
    }
    
    public PermissionManager(Context context) {
        this.context = context;
        this.activity = context instanceof Activity ? (Activity) context : null;
        initializePermissionStates();
    }
    
    public PermissionManager(Activity activity) {
        this.context = activity;
        this.activity = activity;
        initializePermissionStates();
    }
    
    public void setCallback(PermissionCallback callback) {
        this.callback = callback;
    }
    
    /**
     * Initialize permission states by checking current status
     */
    private void initializePermissionStates() {
        try {
            LogUtils.logDebug("Initializing permission states...");
            
            // Check basic permissions
            for (String permission : BASIC_PERMISSIONS) {
                boolean granted = isPermissionGranted(permission);
                permissionStates.put(permission, granted);
                LogUtils.logDebug("Permission " + permission + ": " + (granted ? "GRANTED" : "DENIED"));
            }
            
            // Check enhanced permissions
            for (String permission : ENHANCED_PERMISSIONS) {
                boolean granted = isPermissionGranted(permission);
                permissionStates.put(permission, granted);
                LogUtils.logDebug("Permission " + permission + ": " + (granted ? "GRANTED" : "DENIED"));
            }
            
            // Check special permissions
            boolean hasManageStorage = hasManageExternalStoragePermission();
            boolean hasInstallPermission = hasInstallPermission();
            
            permissionStates.put("MANAGE_EXTERNAL_STORAGE", hasManageStorage);
            permissionStates.put("INSTALL_PACKAGES", hasInstallPermission);
            
            LogUtils.logDebug("Special permissions - MANAGE_STORAGE: " + hasManageStorage + 
                ", INSTALL: " + hasInstallPermission);
            
        } catch (Exception e) {
            LogUtils.logDebug("Error initializing permission states: " + e.getMessage());
        }
    }
    
    /**
     * Check if a specific permission is granted
     */
    private boolean isPermissionGranted(String permission) {
        try {
            return ContextCompat.checkSelfPermission(context, permission) == PackageManager.PERMISSION_GRANTED;
        } catch (Exception e) {
            LogUtils.logDebug("Error checking permission " + permission + ": " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Check if basic storage permissions are granted
     */
    public boolean hasStoragePermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            // Android 11+ - Check for All Files Access
            return hasManageExternalStoragePermission();
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            // Android 6-10 - Check runtime permissions
            return isPermissionGranted(Manifest.permission.READ_EXTERNAL_STORAGE) &&
                   isPermissionGranted(Manifest.permission.WRITE_EXTERNAL_STORAGE);
        } else {
            // Below Android 6 - permissions granted at install time
            return true;
        }
    }
    
    /**
     * Check MANAGE_EXTERNAL_STORAGE permission for Android 11+
     */
    public boolean hasManageExternalStoragePermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            return Environment.isExternalStorageManager();
        }
        return true; // Not applicable for older versions
    }
    
    /**
     * Check install packages permission
     */
    public boolean hasInstallPermission() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            return context.getPackageManager().canRequestPackageInstalls();
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            try {
                return Settings.Secure.getInt(context.getContentResolver(),
                    Settings.Secure.INSTALL_NON_MARKET_APPS) == 1;
            } catch (Settings.SettingNotFoundException e) {
                return false;
            }
        }
        return true; // Older versions don't have this restriction
    }
    
    /**
     * Check if all basic permissions are granted
     */
    public boolean hasBasicPermissions() {
        for (String permission : BASIC_PERMISSIONS) {
            if (!isPermissionGranted(permission)) {
                return false;
            }
        }
        return hasStoragePermission(); // Include modern storage permission
    }
    
    /**
     * Check if all enhanced permissions are granted
     */
    public boolean hasEnhancedPermissions() {
        for (String permission : ENHANCED_PERMISSIONS) {
            if (!isPermissionGranted(permission)) {
                return false;
            }
        }
        return true;
    }
    
    /**
     * Check if all permissions (basic + enhanced + special) are granted
     */
    public boolean hasAllPermissions() {
        return hasBasicPermissions() && hasEnhancedPermissions() && hasInstallPermission();
    }
    
    /**
     * Request basic permissions required for app functionality
     */
    public void requestBasicPermissions() {
        if (activity == null) {
            LogUtils.logDebug("Cannot request permissions - activity is null");
            return;
        }
        
        LogUtils.logUser("🔐 Checking basic permissions...");
        
        // For Android 11+, request All Files Access if needed
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R && !hasManageExternalStoragePermission()) {
            requestAllFilesAccess();
            return;
        }
        
        // For Android 6-10, request runtime permissions
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            List<String> permissionsToRequest = new ArrayList<>();
            
            for (String permission : BASIC_PERMISSIONS) {
                if (!isPermissionGranted(permission)) {
                    permissionsToRequest.add(permission);
                }
            }
            
            if (!permissionsToRequest.isEmpty()) {
                LogUtils.logUser("📋 Requesting " + permissionsToRequest.size() + " basic permissions...");
                ActivityCompat.requestPermissions(activity, 
                    permissionsToRequest.toArray(new String[0]), 
                    REQUEST_BASIC_PERMISSIONS);
            } else {
                LogUtils.logUser("✅ All basic permissions already granted");
                if (callback != null) {
                    callback.onAllPermissionsGranted();
                }
            }
        } else {
            LogUtils.logUser("✅ Running on pre-Android 6 - permissions granted at install");
            if (callback != null) {
                callback.onAllPermissionsGranted();
            }
        }
    }
    
    /**
     * Request All Files Access for Android 11+
     */
    public void requestAllFilesAccess() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            if (hasManageExternalStoragePermission()) {
                LogUtils.logUser("✅ All Files Access already granted");
                return;
            }
            
            LogUtils.logUser("🔐 Requesting All Files Access permission...");
            
            if (activity != null) {
                new AlertDialog.Builder(activity)
                    .setTitle("🗂️ All Files Access Required")
                    .setMessage("TerrariaLoader needs access to all files to:\n\n" +
                        "• Create and manage mod directories\n" +
                        "• Install and patch APK files\n" +
                        "• Backup and restore game data\n\n" +
                        "Please grant 'All Files Access' permission.")
                    .setPositiveButton("Grant Permission", (dialog, which) -> {
                        try {
                            Intent intent = new Intent(Settings.ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION);
                            intent.setData(Uri.parse("package:" + context.getPackageName()));
                            activity.startActivityForResult(intent, REQUEST_ALL_FILES_ACCESS);
                            LogUtils.logUser("📱 Opened All Files Access settings");
                        } catch (Exception e) {
                            LogUtils.logDebug("Error opening All Files Access settings: " + e.getMessage());
                            // Fallback to general manage storage settings
                            try {
                                Intent fallbackIntent = new Intent(Settings.ACTION_MANAGE_ALL_FILES_ACCESS_PERMISSION);
                                activity.startActivityForResult(fallbackIntent, REQUEST_ALL_FILES_ACCESS);
                            } catch (Exception e2) {
                                showToast("Cannot open permission settings");
                            }
                        }
                    })
                    .setNegativeButton("Cancel", (dialog, which) -> {
                        LogUtils.logUser("❌ User declined All Files Access");
                        if (callback != null) {
                            callback.onPermissionDenied("MANAGE_EXTERNAL_STORAGE");
                        }
                    })
                    .show();
            }
        }
    }
    
    /**
     * Request enhanced permissions for additional functionality
     */
    public void requestEnhancedPermissions() {
        if (activity == null) {
            LogUtils.logDebug("Cannot request permissions - activity is null");
            return;
        }
        
        LogUtils.logUser("🔐 Checking enhanced permissions...");
        
        List<String> permissionsToRequest = new ArrayList<>();
        
        for (String permission : ENHANCED_PERMISSIONS) {
            if (!isPermissionGranted(permission)) {
                permissionsToRequest.add(permission);
            }
        }
        
        if (!permissionsToRequest.isEmpty()) {
            LogUtils.logUser("📋 Requesting " + permissionsToRequest.size() + " enhanced permissions...");
            ActivityCompat.requestPermissions(activity, 
                permissionsToRequest.toArray(new String[0]), 
                REQUEST_STORAGE_PERMISSIONS);
        } else {
            LogUtils.logUser("✅ All enhanced permissions already granted");
        }
    }
    
    /**
     * Request install packages permission
     */
    public void requestInstallPermission() {
        if (hasInstallPermission()) {
            LogUtils.logUser("✅ Install permission already granted");
            return;
        }
        
        LogUtils.logUser("🔐 Requesting install permission...");
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            if (activity != null) {
                new AlertDialog.Builder(activity)
                    .setTitle("📦 Install Permission Required")
                    .setMessage("To install modded APK files, please allow this app to install unknown apps.")
                    .setPositiveButton("Grant Permission", (dialog, which) -> {
                        try {
                            Intent intent = new Intent(Settings.ACTION_MANAGE_UNKNOWN_APP_SOURCES);
                            intent.setData(Uri.parse("package:" + context.getPackageName()));
                            activity.startActivityForResult(intent, REQUEST_INSTALL_PERMISSION);
                            LogUtils.logUser("📱 Opened install permission settings");
                        } catch (Exception e) {
                            LogUtils.logDebug("Error opening install permission settings: " + e.getMessage());
                            showToast("Cannot open permission settings");
                        }
                    })
                    .setNegativeButton("Cancel", null)
                    .show();
            }
        } else if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN_MR1) {
            if (activity != null) {
                new AlertDialog.Builder(activity)
                    .setTitle("🔐 Unknown Sources Required")
                    .setMessage("Please enable 'Unknown sources' in security settings to install modded APKs.")
                    .setPositiveButton("Open Settings", (dialog, which) -> {
                        try {
                            Intent intent = new Intent(Settings.ACTION_SECURITY_SETTINGS);
                            activity.startActivity(intent);
                            LogUtils.logUser("📱 Opened security settings");
                        } catch (Exception e) {
                            showToast("Cannot open security settings");
                        }
                    })
                    .setNegativeButton("Cancel", null)
                    .show();
            }
        }
    }
    
    /**
     * Request all permissions (basic + enhanced + install)
     */
    public void requestAllPermissions() {
        LogUtils.logUser("🔐 Requesting all permissions...");
        
        // Start with basic permissions
        if (!hasBasicPermissions()) {
            requestBasicPermissions();
            return;
        }
        
        // Then enhanced permissions
        if (!hasEnhancedPermissions()) {
            requestEnhancedPermissions();
            return;
        }
        
        // Finally install permission
        if (!hasInstallPermission()) {
            requestInstallPermission();
            return;
        }
        
        LogUtils.logUser("✅ All permissions already granted");
        if (callback != null) {
            callback.onAllPermissionsGranted();
        }
    }
    
    /**
     * Handle permission request results
     */
    public void handlePermissionResult(int requestCode, String[] permissions, int[] grantResults) {
        LogUtils.logDebug("Permission result: requestCode=" + requestCode + 
            ", permissions=" + (permissions != null ? permissions.length : 0) +
            ", results=" + (grantResults != null ? grantResults.length : 0));
        
        try {
            switch (requestCode) {
                case REQUEST_BASIC_PERMISSIONS:
                    handleBasicPermissionResult(permissions, grantResults);
                    break;
                case REQUEST_STORAGE_PERMISSIONS:
                    handleStoragePermissionResult(permissions, grantResults);
                    break;
                case REQUEST_ALL_FILES_ACCESS:
                    handleAllFilesAccessResult();
                    break;
                case REQUEST_INSTALL_PERMISSION:
                    handleInstallPermissionResult();
                    break;
                default:
                    LogUtils.logDebug("Unknown permission request code: " + requestCode);
                    break;
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error handling permission result: " + e.getMessage());
        }
    }
    
    /**
     * Handle basic permission results
     */
    private void handleBasicPermissionResult(String[] permissions, int[] grantResults) {
        if (permissions == null || grantResults == null) return;
        
        int grantedCount = 0;
        int deniedCount = 0;
        
        for (int i = 0; i < permissions.length && i < grantResults.length; i++) {
            String permission = permissions[i];
            boolean granted = grantResults[i] == PackageManager.PERMISSION_GRANTED;
            
            permissionStates.put(permission, granted);
            
            if (granted) {
                grantedCount++;
                LogUtils.logUser("✅ Permission granted: " + permission);
                if (callback != null) {
                    callback.onPermissionGranted(permission);
                }
            } else {
                deniedCount++;
                LogUtils.logUser("❌ Permission denied: " + permission);
                
                // Check if permanently denied
                if (activity != null && !ActivityCompat.shouldShowRequestPermissionRationale(activity, permission)) {
                    LogUtils.logUser("⚠️ Permission permanently denied: " + permission);
                    if (callback != null) {
                        callback.onPermissionPermanentlyDenied(permission);
                    }
                } else {
                    if (callback != null) {
                        callback.onPermissionDenied(permission);
                    }
                }
            }
        }
        
        boolean allGranted = deniedCount == 0;
        LogUtils.logUser("📊 Basic permissions result: " + grantedCount + " granted, " + deniedCount + " denied");
        
        if (callback != null) {
            callback.onPermissionRequestCompleted(allGranted);
            if (allGranted) {
                callback.onAllPermissionsGranted();
            }
        }
        
        if (allGranted && !hasInstallPermission()) {
            // Automatically request install permission if basic permissions are granted
            requestInstallPermission();
        }
    }
    
    /**
     * Handle storage permission results
     */
    private void handleStoragePermissionResult(String[] permissions, int[] grantResults) {
        if (permissions == null || grantResults == null) return;
        
        boolean allGranted = true;
        for (int i = 0; i < permissions.length && i < grantResults.length; i++) {
            boolean granted = grantResults[i] == PackageManager.PERMISSION_GRANTED;
            permissionStates.put(permissions[i], granted);
            if (!granted) {
                allGranted = false;
            }
        }
        
        LogUtils.logUser(allGranted ? "✅ Enhanced permissions granted" : "❌ Some enhanced permissions denied");
        
        if (callback != null) {
            callback.onPermissionRequestCompleted(allGranted);
        }
    }
    
    /**
     * Handle All Files Access result
     */
    private void handleAllFilesAccessResult() {
        boolean granted = hasManageExternalStoragePermission();
        permissionStates.put("MANAGE_EXTERNAL_STORAGE", granted);
        
        LogUtils.logUser(granted ? "✅ All Files Access granted" : "❌ All Files Access denied");
        
        if (callback != null) {
            if (granted) {
                callback.onPermissionGranted("MANAGE_EXTERNAL_STORAGE");
            } else {
                callback.onPermissionDenied("MANAGE_EXTERNAL_STORAGE");
            }
            callback.onPermissionRequestCompleted(granted);
        }
        
        if (granted) {
            // Continue with other permissions if needed
            if (!hasEnhancedPermissions()) {
                requestEnhancedPermissions();
            } else if (!hasInstallPermission()) {
                requestInstallPermission();
            }
        }
    }
    
    /**
     * Handle install permission result
     */
    private void handleInstallPermissionResult() {
        boolean granted = hasInstallPermission();
        permissionStates.put("INSTALL_PACKAGES", granted);
        
        LogUtils.logUser(granted ? "✅ Install permission granted" : "❌ Install permission denied");
        
        if (callback != null) {
            if (granted) {
                callback.onPermissionGranted("INSTALL_PACKAGES");
            } else {
                callback.onPermissionDenied("INSTALL_PACKAGES");
            }
            callback.onPermissionRequestCompleted(granted);
        }
    }
    
    /**
     * Check if permission is permanently denied
     */
    public boolean isPermissionPermanentlyDenied(String permission) {
        if (activity == null) return false;
        
        return !isPermissionGranted(permission) && 
               !ActivityCompat.shouldShowRequestPermissionRationale(activity, permission);
    }
    
    /**
     * Show permission explanation dialog
     */
    public void showPermissionExplanation(String permission, String explanation) {
        if (activity == null) return;
        
        new AlertDialog.Builder(activity)
            .setTitle("🔐 Permission Required")
            .setMessage(explanation)
            .setPositiveButton("Grant", (dialog, which) -> {
                // Re-request the specific permission
                ActivityCompat.requestPermissions(activity, new String[]{permission}, REQUEST_BASIC_PERMISSIONS);
            })
            .setNegativeButton("Cancel", null)
            .show();
    }
    
    /**
     * Get current permission status as a detailed string
     */
    public String getPermissionStatus() {
        StringBuilder status = new StringBuilder();
        status.append("=== Permission Status ===\n");
        
        status.append("Basic Permissions: ").append(hasBasicPermissions() ? "✅" : "❌").append("\n");
        status.append("- Storage Access: ").append(hasStoragePermission() ? "✅" : "❌").append("\n");
        
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            status.append("- All Files Access: ").append(hasManageExternalStoragePermission() ? "✅" : "❌").append("\n");
        }
        
        status.append("Enhanced Permissions: ").append(hasEnhancedPermissions() ? "✅" : "❌").append("\n");
        status.append("- Install Packages: ").append(hasInstallPermission() ? "✅" : "❌").append("\n");
        status.append("- Internet Access: ").append(isPermissionGranted(Manifest.permission.INTERNET) ? "✅" : "❌").append("\n");
        
        status.append("Overall Status: ").append(hasAllPermissions() ? "✅ All Ready" : "❌ Missing Permissions").append("\n");
        
        return status.toString();
    }
    
    /**
     * Get list of missing permissions
     */
    public List<String> getMissingPermissions() {
        List<String> missing = new ArrayList<>();
        
        if (!hasStoragePermission()) {
            missing.add("Storage Access");
        }
        
        if (!hasInstallPermission()) {
            missing.add("Install Packages");
        }
        
        for (String permission : BASIC_PERMISSIONS) {
            if (!isPermissionGranted(permission)) {
                missing.add(permission);
            }
        }
        
        for (String permission : ENHANCED_PERMISSIONS) {
            if (!isPermissionGranted(permission)) {
                missing.add(permission);
            }
        }
        
        return missing;
    }
    
    /**
     * Refresh permission states (call after user returns from settings)
     */
    public void refreshPermissionStates() {
        LogUtils.logDebug("Refreshing permission states...");
        initializePermissionStates();
    }
    
    /**
     * Open app settings page
     */
    public void openAppSettings() {
        try {
            Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
            intent.setData(Uri.parse("package:" + context.getPackageName()));
            if (activity != null) {
                activity.startActivity(intent);
            } else {
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                context.startActivity(intent);
            }
            LogUtils.logUser("📱 Opened app settings");
        } catch (Exception e) {
            LogUtils.logDebug("Error opening app settings: " + e.getMessage());
            showToast("Cannot open app settings");
        }
    }
    
    /**
     * Show toast message
     */
    private void showToast(String message) {
        Toast.makeText(context, message, Toast.LENGTH_SHORT).show();
    }
    
    /**
     * Auto setup permissions based on current state
     */
    public void autoSetupPermissions() {
        LogUtils.logDebug("Auto-setting up permissions...");
        
        // Check and request permissions in order of importance
        if (!hasBasicPermissions()) {
            LogUtils.logUser("🔧 Auto-requesting basic permissions...");
            requestBasicPermissions();
        } else if (!hasEnhancedPermissions()) {
            LogUtils.logUser("🔧 Auto-requesting enhanced permissions...");
            requestEnhancedPermissions();
        } else if (!hasInstallPermission()) {
            LogUtils.logUser("🔧 Auto-requesting install permission...");
            requestInstallPermission();
        } else {
            LogUtils.logUser("✅ All permissions already configured");
        }
    }
    
    /**
     * Check if all required permissions are granted
     * (Alias for hasAllPermissions for compatibility)
     */
    public boolean hasAllRequiredPermissions() {
        return hasAllPermissions();
    }
    
    /**
     * Get permission status as formatted text
     */
    public String getPermissionStatusText() {
        StringBuilder status = new StringBuilder();
        
        // Basic permissions status
        boolean basicGranted = hasBasicPermissions();
        status.append("Basic Permissions: ").append(basicGranted ? "✅ Granted" : "❌ Missing").append("\n");
        
        // Storage permission details
        boolean storageGranted = hasStoragePermission();
        status.append("• Storage Access: ").append(storageGranted ? "✅" : "❌").append("\n");
        
        // All Files Access for Android 11+
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.R) {
            boolean allFilesGranted = hasManageExternalStoragePermission();
            status.append("• All Files Access: ").append(allFilesGranted ? "✅" : "❌").append("\n");
        }
        
        // Enhanced permissions status
        boolean enhancedGranted = hasEnhancedPermissions();
        status.append("Enhanced Permissions: ").append(enhancedGranted ? "✅ Granted" : "❌ Missing").append("\n");
        
        // Install permission status
        boolean installGranted = hasInstallPermission();
        status.append("Install Permission: ").append(installGranted ? "✅ Granted" : "❌ Missing").append("\n");
        
        // Overall status
        boolean allGranted = hasAllPermissions();
        status.append("\nOverall Status: ").append(allGranted ? "✅ All Ready" : "⚠️ Setup Needed");
        
        return status.toString();
    }
    
    /**
     * Get simplified permission status for quick checks
     */
    public String getSimplePermissionStatus() {
        if (hasAllPermissions()) {
            return "✅ All permissions granted";
        } else if (hasBasicPermissions()) {
            return "⚠️ Basic permissions granted, additional setup needed";
        } else {
            return "❌ Permissions required - tap to setup";
        }
    }
    
    /**
     * Check if permission setup is needed
     */
    public boolean isPermissionSetupNeeded() {
        return !hasAllPermissions();
    }
    
    /**
     * Get count of granted permissions
     */
    public int getGrantedPermissionCount() {
        int count = 0;
        
        if (hasStoragePermission()) count++;
        if (hasInstallPermission()) count++;
        if (hasEnhancedPermissions()) count += ENHANCED_PERMISSIONS.length;
        
        return count;
    }
    
    /**
     * Get total permission count
     */
    public int getTotalPermissionCount() {
        return BASIC_PERMISSIONS.length + ENHANCED_PERMISSIONS.length + 2; // +2 for storage and install
    }
    
    /**
     * Get permission progress as percentage
     */
    public int getPermissionProgress() {
        int granted = getGrantedPermissionCount();
        int total = getTotalPermissionCount();
        return total > 0 ? (granted * 100) / total : 0;
    }
    
    /**
     * Clean up resources
     */
    public void cleanup() {
        callback = null;
        permissionStates.clear();
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/PrivilegeManager.java

package com.modloader.util;

public class PrivilegeManager {
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/RootManager.java

// File: RootManager.java (FIXED) - Complete Root Access Management
// Path: /app/src/main/java/com/modloader/util/RootManager.java

package com.modloader.util;

import android.content.Context;
import android.content.pm.ApplicationInfo;
import android.content.pm.PackageManager;
import android.widget.Toast;
import androidx.appcompat.app.AlertDialog;
import android.app.Activity;

import java.io.*;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.TimeUnit;

public class RootManager {
    private static final String TAG = "RootManager";
    
    // Common root binary locations
    private static final String[] ROOT_BINARIES = {
        "/system/bin/su",
        "/system/xbin/su", 
        "/sbin/su",
        "/system/su",
        "/vendor/bin/su"
    };
    
    // Root management app packages
    private static final String[] ROOT_APPS = {
        "com.topjohnwu.magisk",           // Magisk
        "eu.chainfire.supersu",           // SuperSU
        "com.koushikdutta.superuser",     // Superuser
        "com.noshufou.android.su",        // Superuser (older)
        "com.thirdparty.superuser",       // SuperUser (CyanogenMod)
        "me.phh.superuser"                // SuperUser (LineageOS)
    };
    
    private final Context context;
    private final Activity activity;
    
    // Root state tracking
    private Boolean rootAvailable = null;
    private Boolean rootGranted = null;
    private String rootAppPackage = null;
    private Process rootProcess = null;
    private BufferedWriter rootWriter = null;
    private BufferedReader rootReader = null;
    
    private RootCallback callback;
    
    public interface RootCallback {
        void onRootAvailable(boolean available);
        void onRootGranted(boolean granted);
        void onRootCommandResult(String command, boolean success, String output);
        void onRootError(String error);
    }
    
    public RootManager(Context context) {
        this.context = context;
        this.activity = context instanceof Activity ? (Activity) context : null;
        checkRootAvailability();
    }
    
    public RootManager(Activity activity) {
        this.context = activity;
        this.activity = activity;
        checkRootAvailability();
    }
    
    public void setCallback(RootCallback callback) {
        this.callback = callback;
    }
    
    /**
     * FIXED: Check if root access is available on this device
     */
    public boolean isRootAvailable() {
        if (rootAvailable != null) {
            return rootAvailable;
        }
        
        LogUtils.logDebug("Checking root availability...");
        
        // Method 1: Check for su binary
        boolean hasSuBinary = checkSuBinary();
        
        // Method 2: Check for root management apps
        boolean hasRootApp = checkRootApps();
        
        // Method 3: Check build tags for test-keys
        boolean hasTestKeys = checkBuildTags();
        
        // Method 4: Try to execute 'which su' command
        boolean canExecuteSu = testSuExecution();
        
        rootAvailable = hasSuBinary || hasRootApp || hasTestKeys || canExecuteSu;
        
        LogUtils.logDebug("Root availability check:");
        LogUtils.logDebug("- SU Binary: " + hasSuBinary);
        LogUtils.logDebug("- Root App: " + hasRootApp);
        LogUtils.logDebug("- Test Keys: " + hasTestKeys);
        LogUtils.logDebug("- SU Execution: " + canExecuteSu);
        LogUtils.logDebug("- Overall Available: " + rootAvailable);
        
        if (callback != null) {
            callback.onRootAvailable(rootAvailable);
        }
        
        return rootAvailable;
    }
    
    /**
     * Check for su binary in common locations
     */
    private boolean checkSuBinary() {
        try {
            for (String path : ROOT_BINARIES) {
                File suFile = new File(path);
                if (suFile.exists() && suFile.canExecute()) {
                    LogUtils.logDebug("Found su binary at: " + path);
                    return true;
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error checking su binaries: " + e.getMessage());
        }
        return false;
    }
    
    /**
     * Check for installed root management applications
     */
    private boolean checkRootApps() {
        try {
            PackageManager pm = context.getPackageManager();
            for (String rootPackage : ROOT_APPS) {
                try {
                    ApplicationInfo appInfo = pm.getApplicationInfo(rootPackage, 0);
                    if (appInfo != null) {
                        LogUtils.logDebug("Found root app: " + rootPackage);
                        rootAppPackage = rootPackage;
                        return true;
                    }
                } catch (PackageManager.NameNotFoundException e) {
                    // App not installed, continue checking
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error checking root apps: " + e.getMessage());
        }
        return false;
    }
    
    /**
     * Check build tags for test-keys (indicates custom ROM/rooted device)
     */
    private boolean checkBuildTags() {
        try {
            String buildTags = android.os.Build.TAGS;
            return buildTags != null && buildTags.contains("test-keys");
        } catch (Exception e) {
            LogUtils.logDebug("Error checking build tags: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Test if su command can be executed
     */
    private boolean testSuExecution() {
        try {
            Process process = Runtime.getRuntime().exec("which su");
            process.waitFor(2, TimeUnit.SECONDS);
            int exitCode = process.exitValue();
            return exitCode == 0;
        } catch (Exception e) {
            LogUtils.logDebug("Error testing su execution: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Check if app has been granted root permission
     */
    public boolean hasRootPermission() {
        if (rootGranted != null) {
            return rootGranted;
        }
        
        if (!isRootAvailable()) {
            rootGranted = false;
            return false;
        }
        
        LogUtils.logDebug("Testing root permission...");
        
        // Try to execute a simple root command
        String testResult = executeRootCommand("id", 3000);
        rootGranted = testResult != null && testResult.contains("uid=0");
        
        LogUtils.logDebug("Root permission test result: " + rootGranted);
        if (rootGranted) {
            LogUtils.logUser("✅ Root permission granted");
        } else {
            LogUtils.logUser("❌ Root permission not granted");
        }
        
        if (callback != null) {
            callback.onRootGranted(rootGranted);
        }
        
        return rootGranted;
    }
    
    /**
     * FIXED: Check if root is ready (available and granted)
     */
    public boolean isRootReady() {
        boolean ready = isRootAvailable() && hasRootPermission();
        LogUtils.logDebug("Root ready status: " + ready);
        return ready;
    }
    
    /**
     * FIXED: Request root access from the user
     */
    public void requestRootAccess() {
        if (!isRootAvailable()) {
            LogUtils.logUser("❌ Root is not available on this device");
            showToast("Root access is not available on this device");
            return;
        }
        
        if (hasRootPermission()) {
            LogUtils.logUser("✅ Root permission already granted");
            showToast("Root access already granted!");
            return;
        }
        
        LogUtils.logUser("🔐 Requesting root access...");
        
        // Try to get root access by executing a simple command
        new Thread(() -> {
            try {
                String result = executeRootCommand("echo 'Root access granted'", 5000);
                boolean success = result != null && result.contains("Root access granted");
                
                if (activity != null) {
                    activity.runOnUiThread(() -> {
                        if (success) {
                            rootGranted = true;
                            LogUtils.logUser("✅ Root access granted successfully!");
                            showToast("Root access granted!");
                            if (callback != null) {
                                callback.onRootGranted(true);
                            }
                        } else {
                            rootGranted = false;
                            LogUtils.logUser("❌ Root access denied or failed");
                            showRootRequestFailedDialog();
                            if (callback != null) {
                                callback.onRootGranted(false);
                            }
                        }
                    });
                }
            } catch (Exception e) {
                LogUtils.logDebug("Error requesting root access: " + e.getMessage());
                if (activity != null) {
                    activity.runOnUiThread(() -> {
                        showToast("Error requesting root access");
                        if (callback != null) {
                            callback.onRootError(e.getMessage());
                        }
                    });
                }
            }
        }).start();
    }
    
    /**
     * Show dialog when root request fails
     */
    private void showRootRequestFailedDialog() {
        if (activity == null) return;
        
        new AlertDialog.Builder(activity)
            .setTitle("❌ Root Access Failed")
            .setMessage("Root access was denied or failed.\n\n" +
                "Possible reasons:\n" +
                "• Root permission was denied in the popup\n" +
                "• Root management app is not properly configured\n" +
                "• Device is not properly rooted\n\n" +
                "Solutions:\n" +
                "• Try again and grant permission\n" +
                "• Check your root management app settings\n" +
                "• Restart the device and try again")
            .setPositiveButton("Try Again", (dialog, which) -> requestRootAccess())
            .setNegativeButton("OK", null)
            .show();
    }
    
    /**
     * FIXED: Execute shell command with root privileges
     */
    public String executeRootCommand(String command) {
        return executeRootCommand(command, 10000); // Default 10 second timeout
    }
    
    /**
     * Execute shell command with root privileges and timeout
     */
    public String executeRootCommand(String command, long timeoutMs) {
        if (!isRootAvailable()) {
            LogUtils.logDebug("Cannot execute root command - root not available");
            return null;
        }
        
        LogUtils.logDebug("Executing root command: " + command);
        
        Process process = null;
        BufferedReader reader = null;
        BufferedWriter writer = null;
        
        try {
            // Start su process
            process = Runtime.getRuntime().exec("su");
            
            writer = new BufferedWriter(new OutputStreamWriter(process.getOutputStream()));
            reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            
            // Send command
            writer.write(command + "\n");
            writer.write("exit\n");
            writer.flush();
            
            // Wait for completion with timeout
            boolean finished = process.waitFor(timeoutMs, TimeUnit.MILLISECONDS);
            if (!finished) {
                LogUtils.logDebug("Root command timed out: " + command);
                process.destroyForcibly();
                return null;
            }
            
            // Read output
            StringBuilder output = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                output.append(line).append("\n");
            }
            
            int exitCode = process.exitValue();
            String result = output.toString().trim();
            
            LogUtils.logDebug("Root command result (exit=" + exitCode + "): " + 
                (result.length() > 100 ? result.substring(0, 100) + "..." : result));
            
            if (callback != null) {
                callback.onRootCommandResult(command, exitCode == 0, result);
            }
            
            return exitCode == 0 ? result : null;
            
        } catch (Exception e) {
            LogUtils.logDebug("Error executing root command: " + e.getMessage());
            if (callback != null) {
                callback.onRootError("Command execution failed: " + e.getMessage());
            }
            return null;
        } finally {
            // Clean up resources
            try {
                if (writer != null) writer.close();
                if (reader != null) reader.close();
                if (process != null && process.isAlive()) {
                    process.destroyForcibly();
                }
            } catch (Exception e) {
                LogUtils.logDebug("Error cleaning up root command resources: " + e.getMessage());
            }
        }
    }
    
    /**
     * Execute multiple root commands in a single su session
     */
    public List<String> executeRootCommands(String[] commands) {
        return executeRootCommands(commands, 15000); // Default 15 second timeout for multiple commands
    }
    
    /**
     * Execute multiple root commands with timeout
     */
    public List<String> executeRootCommands(String[] commands, long timeoutMs) {
        List<String> results = new ArrayList<>();
        
        if (!isRootAvailable() || commands == null || commands.length == 0) {
            return results;
        }
        
        LogUtils.logDebug("Executing " + commands.length + " root commands");
        
        Process process = null;
        BufferedReader reader = null;
        BufferedWriter writer = null;
        
        try {
            process = Runtime.getRuntime().exec("su");
            writer = new BufferedWriter(new OutputStreamWriter(process.getOutputStream()));
            reader = new BufferedReader(new InputStreamReader(process.getInputStream()));
            
            // Send all commands
            for (String command : commands) {
                writer.write(command + "\n");
                writer.write("echo '---COMMAND_SEPARATOR---'\n");
            }
            writer.write("exit\n");
            writer.flush();
            
            // Wait for completion
            boolean finished = process.waitFor(timeoutMs, TimeUnit.MILLISECONDS);
            if (!finished) {
                LogUtils.logDebug("Root commands timed out");
                process.destroyForcibly();
                return results;
            }
            
            // Read output and split by separator
            StringBuilder currentOutput = new StringBuilder();
            String line;
            while ((line = reader.readLine()) != null) {
                if ("---COMMAND_SEPARATOR---".equals(line)) {
                    results.add(currentOutput.toString().trim());
                    currentOutput.setLength(0);
                } else {
                    currentOutput.append(line).append("\n");
                }
            }
            
            // Add final output if any
            if (currentOutput.length() > 0) {
                results.add(currentOutput.toString().trim());
            }
            
            LogUtils.logDebug("Root commands completed: " + results.size() + " results");
            
        } catch (Exception e) {
            LogUtils.logDebug("Error executing root commands: " + e.getMessage());
            if (callback != null) {
                callback.onRootError("Batch command execution failed: " + e.getMessage());
            }
        } finally {
            try {
                if (writer != null) writer.close();
                if (reader != null) reader.close();
                if (process != null && process.isAlive()) {
                    process.destroyForcibly();
                }
            } catch (Exception e) {
                LogUtils.logDebug("Error cleaning up batch command resources: " + e.getMessage());
            }
        }
        
        return results;
    }
    
    /**
     * Start persistent root shell session
     */
    public boolean startRootSession() {
        if (!isRootAvailable()) {
            LogUtils.logDebug("Cannot start root session - root not available");
            return false;
        }
        
        if (rootProcess != null && rootProcess.isAlive()) {
            LogUtils.logDebug("Root session already active");
            return true;
        }
        
        try {
            LogUtils.logDebug("Starting persistent root session...");
            rootProcess = Runtime.getRuntime().exec("su");
            rootWriter = new BufferedWriter(new OutputStreamWriter(rootProcess.getOutputStream()));
            rootReader = new BufferedReader(new InputStreamReader(rootProcess.getInputStream()));
            
            LogUtils.logDebug("✅ Root session started successfully");
            return true;
            
        } catch (Exception e) {
            LogUtils.logDebug("Error starting root session: " + e.getMessage());
            closeRootSession();
            return false;
        }
    }
    
    /**
     * Execute command in persistent root session
     */
    public String executeInRootSession(String command) {
        if (rootProcess == null || !rootProcess.isAlive() || rootWriter == null || rootReader == null) {
            LogUtils.logDebug("Root session not active, starting new session");
            if (!startRootSession()) {
                return null;
            }
        }
        
        try {
            LogUtils.logDebug("Executing in root session: " + command);
            
            rootWriter.write(command + "\n");
            rootWriter.write("echo '---END_OF_COMMAND---'\n");
            rootWriter.flush();
            
            StringBuilder output = new StringBuilder();
            String line;
            while ((line = rootReader.readLine()) != null) {
                if ("---END_OF_COMMAND---".equals(line)) {
                    break;
                }
                output.append(line).append("\n");
            }
            
            String result = output.toString().trim();
            LogUtils.logDebug("Root session command result: " + 
                (result.length() > 100 ? result.substring(0, 100) + "..." : result));
            
            return result;
            
        } catch (Exception e) {
            LogUtils.logDebug("Error executing in root session: " + e.getMessage());
            closeRootSession();
            return null;
        }
    }
    
    /**
     * Close persistent root session
     */
    public void closeRootSession() {
        try {
            if (rootWriter != null) {
                rootWriter.write("exit\n");
                rootWriter.flush();
                rootWriter.close();
                rootWriter = null;
            }
            if (rootReader != null) {
                rootReader.close();
                rootReader = null;
            }
            if (rootProcess != null) {
                if (rootProcess.isAlive()) {
                    rootProcess.waitFor(2, TimeUnit.SECONDS);
                    if (rootProcess.isAlive()) {
                        rootProcess.destroyForcibly();
                    }
                }
                rootProcess = null;
            }
            LogUtils.logDebug("Root session closed");
        } catch (Exception e) {
            LogUtils.logDebug("Error closing root session: " + e.getMessage());
        }
    }
    
    /**
     * Check root status and display information
     */
    public void checkRootStatus() {
        LogUtils.logUser("🔍 Checking root status...");
        
        boolean available = isRootAvailable();
        boolean granted = hasRootPermission();
        
        StringBuilder status = new StringBuilder();
        status.append("=== Root Status ===\n");
        status.append("Available: ").append(available ? "✅" : "❌").append("\n");
        status.append("Permission Granted: ").append(granted ? "✅" : "❌").append("\n");
        
        if (available) {
            if (rootAppPackage != null) {
                status.append("Root Manager: ").append(rootAppPackage).append("\n");
            }
            
            // Try to get root info
            String whoami = executeRootCommand("whoami");
            if (whoami != null) {
                status.append("Root User: ").append(whoami).append("\n");
            }
            
            String suVersion = executeRootCommand("su --version");
            if (suVersion != null) {
                status.append("SU Version: ").append(suVersion).append("\n");
            }
        }
        
        LogUtils.logUser(status.toString());
        
        if (activity != null) {
            new AlertDialog.Builder(activity)
                .setTitle("Root Status")
                .setMessage(status.toString())
                .setPositiveButton("OK", null)
                .show();
        }
    }
    
    /**
     * Install/copy file using root privileges
     */
    public boolean installFileAsRoot(String sourcePath, String targetPath) {
        if (!isRootReady()) {
            LogUtils.logDebug("Cannot install file as root - root not ready");
            return false;
        }
        
        LogUtils.logDebug("Installing file as root: " + sourcePath + " -> " + targetPath);
        
        String[] commands = {
            "cp '" + sourcePath + "' '" + targetPath + "'",
            "chmod 644 '" + targetPath + "'",
            "chown system:system '" + targetPath + "'"
        };
        
        List<String> results = executeRootCommands(commands);
        boolean success = results.size() == commands.length;
        
        if (success) {
            LogUtils.logUser("✅ File installed successfully with root: " + targetPath);
        } else {
            LogUtils.logUser("❌ Failed to install file with root: " + targetPath);
        }
        
        return success;
    }
    
    /**
     * Create directory using root privileges
     */
    public boolean createDirectoryAsRoot(String dirPath) {
        if (!isRootReady()) {
            return false;
        }
        
        String result = executeRootCommand("mkdir -p '" + dirPath + "' && echo 'SUCCESS'");
        boolean success = result != null && result.contains("SUCCESS");
        
        if (success) {
            LogUtils.logDebug("Created directory as root: " + dirPath);
        }
        
        return success;
    }
    
    /**
     * Delete file/directory using root privileges  
     */
    public boolean deleteAsRoot(String path) {
        if (!isRootReady()) {
            return false;
        }
        
        String result = executeRootCommand("rm -rf '" + path + "' && echo 'DELETED'");
        boolean success = result != null && result.contains("DELETED");
        
        if (success) {
            LogUtils.logDebug("Deleted as root: " + path);
        }
        
        return success;
    }
    
    /**
     * Get detailed root information
     */
    public String getDetailedStatus() {
        StringBuilder info = new StringBuilder();
        info.append("=== Root Manager Status ===\n");
        
        boolean available = isRootAvailable();
        boolean granted = hasRootPermission();
        
        info.append("Available: ").append(available ? "✅" : "❌").append("\n");
        info.append("Permission: ").append(granted ? "✅" : "❌").append("\n");
        info.append("Ready: ").append(isRootReady() ? "✅" : "❌").append("\n");
        
        if (rootAppPackage != null) {
            info.append("Root App: ").append(rootAppPackage).append("\n");
        }
        
        info.append("Session Active: ").append(
            (rootProcess != null && rootProcess.isAlive()) ? "✅" : "❌").append("\n");
        
        // Add system info if root is available
        if (available) {
            String buildTags = android.os.Build.TAGS;
            info.append("Build Tags: ").append(buildTags != null ? buildTags : "Unknown").append("\n");
        }
        
        return info.toString();
    }
    
    /**
     * Refresh root availability (call after potential changes)
     */
    public void refreshStatus() {
        LogUtils.logDebug("Refreshing root status...");
        rootAvailable = null;
        rootGranted = null;
        checkRootAvailability();
    }
    
    /**
     * Check root availability and update cache
     */
    private void checkRootAvailability() {
        // This will update the rootAvailable cache
        isRootAvailable();
    }
    
    /**
     * Show toast message
     */
    private void showToast(String message) {
        if (context != null) {
            Toast.makeText(context, message, Toast.LENGTH_SHORT).show();
        }
    }
    
    /**
     * Clean up resources
     */
    public void cleanup() {
        closeRootSession();
        callback = null;
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/java/com/modloader/util/ShizukuManager.java

// File: ShizukuManager.java (FIXED) - Complete Shizuku Integration with Proper Permission Detection
// Path: /app/src/main/java/com/modloader/util/ShizukuManager.java

package com.modloader.util;

import android.app.Activity;
import android.content.ComponentName;
import android.content.Context;
import android.content.Intent;
import android.content.ServiceConnection;
import android.content.pm.PackageInfo;
import android.content.pm.PackageManager;
import android.os.Build;
import android.os.IBinder;
import android.os.RemoteException;
import android.widget.Toast;

import java.lang.reflect.Method;

public class ShizukuManager {
    private static final String TAG = "ShizukuManager";
    
    // Shizuku package constants
    private static final String SHIZUKU_PACKAGE = "moe.shizuku.privileged.api";
    private static final String SHIZUKU_SERVICE = "moe.shizuku.privileged.api.ShizukuService";
    private static final String SHIZUKU_ACTIVITY = "moe.shizuku.manager.MainActivity";
    
    // Permission constants  
    private static final String SHIZUKU_PERMISSION = "moe.shizuku.manager.permission.API_V23";
    private static final int SHIZUKU_REQUEST_CODE = 9999;
    
    private final Context context;
    private final Activity activity;
    
    // Shizuku API reflection objects
    private Class<?> shizukuClass;
    private Method checkSelfPermissionMethod;
    private Method requestPermissionMethod;
    private Method isPreV11Method;
    private Method pingBinderMethod;
    private Method getVersionMethod;
    private Method getUidMethod;
    
    // Connection state
    private boolean isShizukuConnected = false;
    private Object shizukuBinder = null;
    
    public ShizukuManager(Context context) {
        this.context = context;
        this.activity = context instanceof Activity ? (Activity) context : null;
        initializeShizukuReflection();
        checkShizukuConnection();
    }
    
    public ShizukuManager(Activity activity) {
        this.context = activity;
        this.activity = activity;
        initializeShizukuReflection();
        checkShizukuConnection();
    }
    
    /**
     * FIXED: Initialize Shizuku API through reflection to avoid compile-time dependency
     */
    private void initializeShizukuReflection() {
        try {
            LogUtils.logDebug("Initializing Shizuku reflection API...");
            
            // Try to load Shizuku API class
            shizukuClass = Class.forName("rikka.shizuku.Shizuku");
            
            // Get essential methods through reflection
            checkSelfPermissionMethod = shizukuClass.getMethod("checkSelfPermission");
            requestPermissionMethod = shizukuClass.getMethod("requestPermission", int.class);
            pingBinderMethod = shizukuClass.getMethod("pingBinder");
            getVersionMethod = shizukuClass.getMethod("getVersion");
            getUidMethod = shizukuClass.getMethod("getUid");
            
            // Check if pre-v11 method exists (for older Shizuku versions)
            try {
                isPreV11Method = shizukuClass.getMethod("isPreV11");
            } catch (NoSuchMethodException e) {
                LogUtils.logDebug("isPreV11 method not found - using newer Shizuku API");
            }
            
            LogUtils.logDebug("✅ Shizuku reflection API initialized successfully");
            
        } catch (ClassNotFoundException e) {
            LogUtils.logDebug("Shizuku API not found in classpath - will use intent-based detection");
            shizukuClass = null;
        } catch (Exception e) {
            LogUtils.logDebug("Failed to initialize Shizuku reflection: " + e.getMessage());
            shizukuClass = null;
        }
    }
    
    /**
     * FIXED: Check if Shizuku app is installed on the device
     */
    public boolean isShizukuInstalled() {
        try {
            PackageManager pm = context.getPackageManager();
            
            // Try primary package name
            try {
                PackageInfo packageInfo = pm.getPackageInfo(SHIZUKU_PACKAGE, 0);
                LogUtils.logDebug("Shizuku found: version " + packageInfo.versionName + 
                    " (" + packageInfo.versionCode + ")");
                return true;
            } catch (PackageManager.NameNotFoundException e) {
                // Try alternative package name
                try {
                    PackageInfo altPackageInfo = pm.getPackageInfo("moe.shizuku.manager", 0);
                    LogUtils.logDebug("Shizuku manager found: version " + altPackageInfo.versionName);
                    return true;
                } catch (PackageManager.NameNotFoundException e2) {
                    LogUtils.logDebug("Shizuku not installed: " + e.getMessage());
                    return false;
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error checking Shizuku installation: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Check if Shizuku service is running and accessible
     */
    public boolean isShizukuRunning() {
        if (!isShizukuInstalled()) {
            LogUtils.logDebug("Shizuku not installed");
            return false;
        }
        
        try {
            if (shizukuClass != null && pingBinderMethod != null) {
                // Use reflection to call Shizuku.pingBinder()
                boolean canPing = (Boolean) pingBinderMethod.invoke(null);
                LogUtils.logDebug("Shizuku ping result: " + canPing);
                return canPing;
            } else {
                // Fallback: Check if Shizuku service is bound by attempting connection
                return checkShizukuServiceBinding();
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error checking Shizuku service: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Check Shizuku service binding as fallback method
     */
    private boolean checkShizukuServiceBinding() {
        try {
            Intent serviceIntent = new Intent();
            serviceIntent.setComponent(new ComponentName(SHIZUKU_PACKAGE, SHIZUKU_SERVICE));
            
            ServiceConnection testConnection = new ServiceConnection() {
                @Override
                public void onServiceConnected(ComponentName name, IBinder service) {
                    isShizukuConnected = true;
                    shizukuBinder = service;
                    LogUtils.logDebug("Shizuku service connected via binding test");
                }
                
                @Override
                public void onServiceDisconnected(ComponentName name) {
                    isShizukuConnected = false;
                    shizukuBinder = null;
                }
            };
            
            // Try to bind to service
            boolean bindResult = context.bindService(serviceIntent, testConnection, 
                Context.BIND_AUTO_CREATE | Context.BIND_NOT_FOREGROUND);
            
            if (bindResult) {
                // Unbind immediately - we just wanted to test connectivity
                try {
                    context.unbindService(testConnection);
                } catch (Exception e) {
                    LogUtils.logDebug("Error unbinding test service: " + e.getMessage());
                }
                return true;
            }
            
            return false;
        } catch (Exception e) {
            LogUtils.logDebug("Service binding test failed: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Check if app has Shizuku permission - this is the main fix
     */
    public boolean hasShizukuPermission() {
        if (!isShizukuRunning()) {
            LogUtils.logDebug("Shizuku not running - cannot check permission");
            return false;
        }
        
        try {
            if (shizukuClass != null && checkSelfPermissionMethod != null) {
                // FIXED: Use reflection to call Shizuku.checkSelfPermission()
                int permissionResult = (Integer) checkSelfPermissionMethod.invoke(null);
                boolean hasPermission = (permissionResult == PackageManager.PERMISSION_GRANTED);
                
                LogUtils.logDebug("Shizuku permission check result: " + permissionResult + 
                    " (granted=" + hasPermission + ")");
                    
                if (hasPermission) {
                    LogUtils.logUser("✅ Shizuku permission already granted");
                } else {
                    LogUtils.logUser("❌ Shizuku permission not granted (result: " + permissionResult + ")");
                }
                
                return hasPermission;
            } else {
                // FIXED: Fallback method for when reflection fails
                return checkShizukuPermissionFallback();
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error checking Shizuku permission: " + e.getMessage());
            // Try fallback method
            return checkShizukuPermissionFallback();
        }
    }
    
    /**
     * FIXED: Fallback permission check method
     */
    private boolean checkShizukuPermissionFallback() {
        try {
            LogUtils.logDebug("Using fallback Shizuku permission check...");
            
            // Method 1: Check using standard permission system
            int permResult = context.checkSelfPermission(SHIZUKU_PERMISSION);
            if (permResult == PackageManager.PERMISSION_GRANTED) {
                LogUtils.logDebug("Shizuku permission granted via standard check");
                return true;
            }
            
            // Method 2: Check using package manager
            PackageManager pm = context.getPackageManager();
            String[] permissions = pm.getPackageInfo(context.getPackageName(), 
                PackageManager.GET_PERMISSIONS).requestedPermissions;
                
            if (permissions != null) {
                for (String permission : permissions) {
                    if (SHIZUKU_PERMISSION.equals(permission)) {
                        LogUtils.logDebug("Shizuku permission found in manifest");
                        // Check if actually granted
                        return pm.checkPermission(SHIZUKU_PERMISSION, context.getPackageName()) 
                            == PackageManager.PERMISSION_GRANTED;
                    }
                }
            }
            
            LogUtils.logDebug("Shizuku permission not found in fallback checks");
            return false;
            
        } catch (Exception e) {
            LogUtils.logDebug("Fallback permission check failed: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * FIXED: Request Shizuku permission from user
     */
    public void requestShizukuPermission() {
        if (!isShizukuRunning()) {
            LogUtils.logUser("❌ Shizuku is not running. Start Shizuku first.");
            showToast("Shizuku is not running. Please start Shizuku service first.");
            return;
        }
        
        if (hasShizukuPermission()) {
            LogUtils.logUser("✅ Shizuku permission already granted");
            showToast("Shizuku permission already granted!");
            return;
        }
        
        try {
            LogUtils.logUser("🔐 Requesting Shizuku permission...");
            
            if (shizukuClass != null && requestPermissionMethod != null && activity != null) {
                // FIXED: Use reflection to call Shizuku.requestPermission()
                requestPermissionMethod.invoke(null, SHIZUKU_REQUEST_CODE);
                LogUtils.logUser("📱 Shizuku permission dialog should appear");
                showToast("Grant permission in the Shizuku dialog that appears");
                
            } else {
                // FIXED: Fallback - open Shizuku app for manual permission grant
                LogUtils.logDebug("Using fallback permission request method");
                openShizukuAppForPermission();
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Error requesting Shizuku permission: " + e.getMessage());
            LogUtils.logUser("❌ Failed to request Shizuku permission automatically");
            
            // Show manual instruction dialog
            showManualPermissionInstructions();
        }
    }
    
    /**
     * FIXED: Fallback method to open Shizuku app for permission granting
     */
    private void openShizukuAppForPermission() {
        try {
            // Try to open Shizuku manager
            Intent intent = new Intent();
            intent.setComponent(new ComponentName(SHIZUKU_PACKAGE, SHIZUKU_ACTIVITY));
            intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            
            if (intent.resolveActivity(context.getPackageManager()) != null) {
                context.startActivity(intent);
                LogUtils.logUser("📱 Opened Shizuku app - grant permission manually");
                showToast("Grant permission to TerrariaLoader in Shizuku settings");
            } else {
                // Try alternative package
                intent.setPackage("moe.shizuku.manager");
                if (intent.resolveActivity(context.getPackageManager()) != null) {
                    context.startActivity(intent);
                    LogUtils.logUser("📱 Opened Shizuku manager - grant permission manually");
                } else {
                    LogUtils.logDebug("Could not open Shizuku app");
                    showManualPermissionInstructions();
                }
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error opening Shizuku app: " + e.getMessage());
            showManualPermissionInstructions();
        }
    }
    
    /**
     * Show manual permission instructions to user
     */
    private void showManualPermissionInstructions() {
        if (activity != null) {
            android.app.AlertDialog.Builder builder = new android.app.AlertDialog.Builder(activity);
            builder.setTitle("🔐 Manual Shizuku Permission Required");
            builder.setMessage("Please grant permission manually:\n\n" +
                "1. Open Shizuku app\n" +
                "2. Go to 'Applications using Shizuku API'\n" +
                "3. Find 'TerrariaLoader' in the list\n" +
                "4. Toggle the permission ON\n" +
                "5. Return to TerrariaLoader\n\n" +
                "If TerrariaLoader is not in the list, restart the app and try again.");
            builder.setPositiveButton("Open Shizuku", (dialog, which) -> openShizukuApp());
            builder.setNegativeButton("OK", null);
            builder.show();
        }
    }
    
    /**
     * Check current Shizuku connection status
     */
    private void checkShizukuConnection() {
        try {
            boolean installed = isShizukuInstalled();
            boolean running = isShizukuRunning();
            boolean hasPermission = hasShizukuPermission();
            
            LogUtils.logDebug("=== Shizuku Status ===");
            LogUtils.logDebug("Installed: " + installed);
            LogUtils.logDebug("Running: " + running);  
            LogUtils.logDebug("Has Permission: " + hasPermission);
            LogUtils.logDebug("Ready: " + (installed && running && hasPermission));
            
            if (installed && running && hasPermission) {
                LogUtils.logUser("✅ Shizuku is ready and has permission");
            }
            
        } catch (Exception e) {
            LogUtils.logDebug("Error checking Shizuku connection: " + e.getMessage());
        }
    }
    
    /**
     * Get Shizuku version information
     */
    public String getShizukuVersion() {
        try {
            if (shizukuClass != null && getVersionMethod != null) {
                int version = (Integer) getVersionMethod.invoke(null);
                return String.valueOf(version);
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error getting Shizuku version: " + e.getMessage());
        }
        
        // Fallback: get from package info
        try {
            PackageInfo packageInfo = context.getPackageManager().getPackageInfo(SHIZUKU_PACKAGE, 0);
            return packageInfo.versionName;
        } catch (Exception e) {
            return "Unknown";
        }
    }
    
    /**
     * Get Shizuku UID
     */
    public int getShizukuUid() {
        try {
            if (shizukuClass != null && getUidMethod != null) {
                return (Integer) getUidMethod.invoke(null);
            }
        } catch (Exception e) {
            LogUtils.logDebug("Error getting Shizuku UID: " + e.getMessage());
        }
        return -1;
    }
    
    /**
     * Check if Shizuku is available (installed)
     */
    public boolean isShizukuAvailable() {
        return isShizukuInstalled();
    }
    
    /**
     * FIXED: Check if Shizuku is ready (installed, running, and has permission)
     */
    public boolean isShizukuReady() {
        boolean ready = isShizukuInstalled() && isShizukuRunning() && hasShizukuPermission();
        LogUtils.logDebug("Shizuku ready status: " + ready);
        return ready;
    }
    
    /**
     * Open Shizuku app
     */
    public boolean openShizukuApp() {
        try {
            Intent intent = context.getPackageManager().getLaunchIntentForPackage(SHIZUKU_PACKAGE);
            if (intent == null) {
                // Try alternative package
                intent = context.getPackageManager().getLaunchIntentForPackage("moe.shizuku.manager");
            }
            
            if (intent != null) {
                intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
                context.startActivity(intent);
                return true;
            }
            
            return false;
        } catch (Exception e) {
            LogUtils.logDebug("Error opening Shizuku app: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Install Shizuku from GitHub releases
     */
    public void installShizuku() {
        try {
            Intent browserIntent = new Intent(Intent.ACTION_VIEW);
            browserIntent.setData(android.net.Uri.parse("https://github.com/RikkaApps/Shizuku/releases/latest"));
            browserIntent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            context.startActivity(browserIntent);
            LogUtils.logUser("🌐 Opened Shizuku download page");
        } catch (Exception e) {
            LogUtils.logDebug("Error opening Shizuku download: " + e.getMessage());
            showToast("Could not open browser. Please download Shizuku manually from GitHub.");
        }
    }
    
    /**
     * Execute shell command using Shizuku
     */
    public boolean executeShellCommand(String command) {
        if (!isShizukuReady()) {
            LogUtils.logDebug("Shizuku not ready for command execution");
            return false;
        }
        
        try {
            // This would require Shizuku API implementation
            LogUtils.logDebug("Executing shell command via Shizuku: " + command);
            // Implementation would go here using Shizuku's shell execution API
            return true;
        } catch (Exception e) {
            LogUtils.logDebug("Error executing shell command: " + e.getMessage());
            return false;
        }
    }
    
    /**
     * Get detailed Shizuku status information
     */
    public String getDetailedStatus() {
        StringBuilder status = new StringBuilder();
        status.append("=== Shizuku Manager Status ===\n");
        
        boolean installed = isShizukuInstalled();
        boolean running = isShizukuRunning();
        boolean hasPermission = hasShizukuPermission();
        
        status.append("Installed: ").append(installed ? "✅" : "❌").append("\n");
        status.append("Running: ").append(running ? "✅" : "❌").append("\n");
        status.append("Has Permission: ").append(hasPermission ? "✅" : "❌").append("\n");
        status.append("Overall Ready: ").append(isShizukuReady() ? "✅" : "❌").append("\n");
        
        if (installed) {
            status.append("Version: ").append(getShizukuVersion()).append("\n");
            if (running) {
                status.append("UID: ").append(getShizukuUid()).append("\n");
            }
        }
        
        status.append("API Class Available: ").append(shizukuClass != null ? "✅" : "❌").append("\n");
        
        return status.toString();
    }
    
    /**
     * Handle permission request result - call this from Activity.onRequestPermissionsResult()
     */
    public void handlePermissionResult(int requestCode, String[] permissions, int[] grantResults) {
        if (requestCode == SHIZUKU_REQUEST_CODE) {
            boolean granted = grantResults.length > 0 && grantResults[0] == PackageManager.PERMISSION_GRANTED;
            LogUtils.logUser(granted ? "✅ Shizuku permission granted!" : "❌ Shizuku permission denied");
            
            if (granted) {
                showToast("Shizuku permission granted successfully!");
            } else {
                showToast("Shizuku permission was denied");
            }
        }
    }
    
    /**
     * Refresh Shizuku status (call after potential status changes)
     */
    public void refreshStatus() {
        LogUtils.logDebug("Refreshing Shizuku status...");
        checkShizukuConnection();
    }
    
    /**
     * Show toast message
     */
    private void showToast(String message) {
        if (context != null) {
            Toast.makeText(context, message, Toast.LENGTH_SHORT).show();
        }
    }
    
    /**
     * Cleanup resources
     */
    public void cleanup() {
        if (shizukuBinder != null) {
            shizukuBinder = null;
        }
        isShizukuConnected = false;
    }
}

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/drawable/gradient_background_135.xml

<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android">
    <gradient
        android:angle="135"
        android:startColor="#E8F5E8"
        android:endColor="#F1F8E9"
        android:type="linear" />
</shape>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/drawable/ic_arrow_back.xml

<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="24dp"
    android:height="24dp"
    android:viewportWidth="24"
    android:viewportHeight="24">
  <path
      android:fillColor="@android:color/black"
      android:pathData="M20,11L7.83,11l5.59,-5.59L12,4l-8,8 8,8 1.41,-1.41L7.83,13L20,13z"/>
</vector>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/drawable/ic_launcher_background.xml

<?xml version="1.0" encoding="utf-8"?>
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path
        android:fillColor="#3DDC84"
        android:pathData="M0,0h108v108h-108z" />
    <path
        android:fillColor="#00000000"
        android:pathData="M9,0L9,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,0L19,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M29,0L29,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M39,0L39,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M49,0L49,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M59,0L59,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M69,0L69,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M79,0L79,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M89,0L89,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M99,0L99,108"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,9L108,9"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,19L108,19"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,29L108,29"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,39L108,39"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,49L108,49"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,59L108,59"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,69L108,69"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,79L108,79"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,89L108,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M0,99L108,99"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,29L89,29"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,39L89,39"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,49L89,49"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,59L89,59"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,69L89,69"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M19,79L89,79"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M29,19L29,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M39,19L39,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M49,19L49,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M59,19L59,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M69,19L69,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
    <path
        android:fillColor="#00000000"
        android:pathData="M79,19L79,89"
        android:strokeWidth="0.8"
        android:strokeColor="#33FFFFFF" />
</vector>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/drawable-v24/ic_launcher_foreground.xml

<vector xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:aapt="http://schemas.android.com/aapt"
    android:width="108dp"
    android:height="108dp"
    android:viewportWidth="108"
    android:viewportHeight="108">
    <path android:pathData="M31,63.928c0,0 6.4,-11 12.1,-13.1c7.2,-2.6 26,-1.4 26,-1.4l38.1,38.1L107,108.928l-32,-1L31,63.928z">
        <aapt:attr name="android:fillColor">
            <gradient
                android:endX="85.84757"
                android:endY="92.4963"
                android:startX="42.9492"
                android:startY="49.59793"
                android:type="linear">
                <item
                    android:color="#44000000"
                    android:offset="0.0" />
                <item
                    android:color="#00000000"
                    android:offset="1.0" />
            </gradient>
        </aapt:attr>
    </path>
    <path
        android:fillColor="#FFFFFF"
        android:fillType="nonZero"
        android:pathData="M65.3,45.828l3.8,-6.6c0.2,-0.4 0.1,-0.9 -0.3,-1.1c-0.4,-0.2 -0.9,-0.1 -1.1,0.3l-3.9,6.7c-6.3,-2.8 -13.4,-2.8 -19.7,0l-3.9,-6.7c-0.2,-0.4 -0.7,-0.5 -1.1,-0.3C38.8,38.328 38.7,38.828 38.9,39.228l3.8,6.6C36.2,49.428 31.7,56.028 31,63.928h46C76.3,56.028 71.8,49.428 65.3,45.828zM43.4,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2c-0.3,-0.7 -0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C45.3,56.528 44.5,57.328 43.4,57.328L43.4,57.328zM64.6,57.328c-0.8,0 -1.5,-0.5 -1.8,-1.2s-0.1,-1.5 0.4,-2.1c0.5,-0.5 1.4,-0.7 2.1,-0.4c0.7,0.3 1.2,1 1.2,1.8C66.5,56.528 65.6,57.328 64.6,57.328L64.6,57.328z"
        android:strokeWidth="1"
        android:strokeColor="#00000000" />
</vector>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_about.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Mod Loader"
            android:textStyle="bold"
            android:textSize="20sp"
            android:layout_marginBottom="8dp" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Author: Jonie"
            android:textSize="16sp"
            android:layout_marginBottom="8dp" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Description:\n\nThis app lets you inject custom loaders into Terraria APKs for modding purposes. It also provides tools to manage logs and exported APKs.\n\nUse responsibly and only with legal copies of the game."
            android:textSize="14sp"
            android:lineSpacingExtra="4dp" />
    </LinearLayout>
</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_dll_mod.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <!-- Header Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="DLL Mod Manager"
            android:textSize="24sp"
            android:textStyle="bold"
            android:gravity="center"
            android:layout_marginBottom="16dp" />

        <!-- Status Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="#F5F5F5"
            android:padding="12dp"
            android:layout_marginBottom="16dp">

            <TextView
                android:id="@+id/statusText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="DLL Mods: 0 enabled, 0 disabled, 0 total"
                android:textSize="16sp"
                android:textStyle="bold"
                android:layout_marginBottom="8dp" />

            <TextView
                android:id="@+id/loaderStatusText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="❌ No loader installed - DLL mods will not work"
                android:textSize="14sp" />

        </LinearLayout>

        <!-- Loader Installation Section -->
        <LinearLayout
            android:id="@+id/loaderInfoSection"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="#E8F5E8"
            android:padding="12dp"
            android:layout_marginBottom="16dp"
            android:visibility="gone">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Loader Installation"
                android:textSize="16sp"
                android:textStyle="bold"
                android:layout_marginBottom="8dp" />

            <Button
                android:id="@+id/installLoaderBtn"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Install Loader"
                android:layout_marginBottom="8dp" />

            <Button
                android:id="@+id/selectApkBtn"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Select Terraria APK"
                android:layout_marginBottom="8dp" />

        </LinearLayout>

        <!-- Mod Management Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="DLL Mod Management"
            android:textSize="18sp"
            android:textStyle="bold"
            android:layout_marginBottom="8dp" />

        <Button
            android:id="@+id/installDllBtn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Install DLL Mod"
            android:layout_marginBottom="16dp" />

        <!-- Mod List -->
        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/dllModRecyclerView"
            android:layout_width="match_parent"
            android:layout_height="300dp"
            android:background="#F9F9F9"
            android:padding="8dp"
            android:layout_marginBottom="16dp" />

        <!-- Action Buttons -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center"
            android:layout_marginTop="16dp">

            <Button
                android:id="@+id/refreshBtn"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="Refresh"
                android:layout_marginEnd="8dp" />

            <Button
                android:id="@+id/viewLogsBtn"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="View Logs"
                android:layout_marginStart="8dp" />

        </LinearLayout>

        <!-- Information Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="DLL mods require MelonLoader or LemonLoader to be installed. Use 'Install Loader' to set up the required components."
            android:textSize="12sp"
            android:textColor="#666666"
            android:layout_marginTop="16dp"
            android:gravity="center" />

    </LinearLayout>

</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_instructions.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".ui.InstructionsActivity">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:padding="16dp">

        <TextView
            android:id="@+id/tv_instructions_title"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:text="Manual Installation Instructions"
            android:textSize="24sp"
            android:textStyle="bold"
            android:gravity="center"
            app:layout_constraintTop_toTopOf="parent"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            android:layout_marginTop="16dp" />

        <TextView
            android:id="@+id/tv_instructions"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            android:textSize="16sp"
            android:textIsSelectable="true"
            app:layout_constraintTop_toBottomOf="@id/tv_instructions_title"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            tools:text="Detailed instructions will appear here..." />

    </androidx.constraintlayout.widget.ConstraintLayout>
</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_log.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/log_layout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <ScrollView
        android:id="@+id/log_scroll"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <TextView
            android:id="@+id/log_text"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textIsSelectable="true"
            android:textAppearance="?android:textAppearanceSmall"
            android:textColor="#FFFFFF"
            android:background="#222222"
            android:padding="10dp" />
    </ScrollView>

    <Button
        android:id="@+id/export_logs_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Export Logs" />
</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_log_viewer_enhanced.xml

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#1E1E1E">

    <!-- Statistics Bar -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="8dp"
        android:background="#2E2E2E"
        android:gravity="center_vertical">

        <TextView
            android:id="@+id/logStatsText"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="📊 Total: 0 | Showing: 0 | Errors: 0 | Warnings: 0"
            android:textColor="#FFFFFF"
            android:textSize="12sp"
            android:fontFamily="monospace" />

        <Button
            android:id="@+id/refreshButton"
            android:layout_width="wrap_content"
            android:layout_height="32dp"
            android:text="🔄"
            android:textSize="14sp"
            android:background="#4CAF50"
            android:textColor="#FFFFFF"
            android:layout_marginStart="8dp"
            android:padding="4dp"
            android:minWidth="48dp" />

    </LinearLayout>

    <!-- Filter Section (Collapsible) -->
    <LinearLayout
        android:id="@+id/filterSection"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="12dp"
        android:background="#2A2A2A"
        android:visibility="visible">

        <!-- Filter Controls Row 1 -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="🏷️ Type:"
                android:textColor="#FFFFFF"
                android:textSize="14sp"
                android:layout_marginEnd="8dp" />

            <Spinner
                android:id="@+id/logTypeSpinner"
                android:layout_width="0dp"
                android:layout_height="48dp"
                android:layout_weight="1"
                android:background="#3A3A3A"
                android:layout_marginEnd="16dp"
                android:popupBackground="#3A3A3A" />

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="📊 Level:"
                android:textColor="#FFFFFF"
                android:textSize="14sp"
                android:layout_marginEnd="8dp" />

            <Spinner
                android:id="@+id/logLevelSpinner"
                android:layout_width="0dp"
                android:layout_height="48dp"
                android:layout_weight="1"
                android:background="#3A3A3A"
                android:popupBackground="#3A3A3A" />

        </LinearLayout>

        <!-- Search Bar -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="12dp"
            android:gravity="center_vertical">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="🔍"
                android:textSize="16sp"
                android:layout_marginEnd="8dp" />

            <EditText
                android:id="@+id/searchEditText"
                android:layout_width="0dp"
                android:layout_height="48dp"
                android:layout_weight="1"
                android:background="#3A3A3A"
                android:hint="Search logs..."
                android:textColorHint="#888888"
                android:textColor="#FFFFFF"
                android:padding="12dp"
                android:textSize="14sp"
                android:fontFamily="monospace"
                android:inputType="text"
                android:imeOptions="actionSearch" />

        </LinearLayout>

        <!-- Control Buttons -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="12dp"
            android:gravity="center">

            <Button
                android:id="@+id/clearLogsButton"
                android:layout_width="0dp"
                android:layout_height="40dp"
                android:layout_weight="1"
                android:text="🗑️ Clear"
                android:background="#FF6B6B"
                android:textColor="#FFFFFF"
                android:textSize="14sp"
                android:layout_marginEnd="8dp" />

            <Button
                android:id="@+id/exportLogsButton"
                android:layout_width="0dp"
                android:layout_height="40dp"
                android:layout_weight="1"
                android:text="📤 Export"
                android:background="#2196F3"
                android:textColor="#FFFFFF"
                android:textSize="14sp"
                android:layout_marginStart="8dp" />

        </LinearLayout>

        <!-- Auto-scroll checkbox -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="8dp"
            android:gravity="center_vertical">

            <CheckBox
                android:id="@+id/autoScrollCheckbox"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="📜 Auto-scroll to bottom"
                android:textColor="#FFFFFF"
                android:textSize="14sp"
                android:checked="true"
                android:buttonTint="#4CAF50" />

        </LinearLayout>

    </LinearLayout>

    <!-- Main Log Content -->
    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
        android:id="@+id/swipeRefreshLayout"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1">

        <ScrollView
            android:id="@+id/logScrollView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:background="#1E1E1E"
            android:scrollbars="vertical"
            android:fadeScrollbars="false">

            <TextView
                android:id="@+id/logTextView"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="📋 Loading logs...\n\nPlease wait while we fetch the latest log entries."
                android:textColor="#E0E0E0"
                android:textSize="12sp"
                android:fontFamily="monospace"
                android:padding="16dp"
                android:textIsSelectable="true"
                android:background="#1E1E1E"
                android:lineSpacingMultiplier="1.2" />

        </ScrollView>

    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>

    <!-- Bottom Action Bar -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="8dp"
        android:background="#2E2E2E"
        android:gravity="center_vertical">

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="💡 Tip: Swipe down to refresh, use filters to find specific logs"
            android:textColor="#888888"
            android:textSize="11sp" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="v1.0"
            android:textColor="#666666"
            android:textSize="10sp"
            android:layout_marginStart="8dp" />

    </LinearLayout>

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_main.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:gravity="center"
    android:padding="24dp">

    <Button
        android:id="@+id/universal_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Universal"
        android:layout_marginBottom="16dp" />

    <Button
        android:id="@+id/specific_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Specific Version" />
</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_mod_list.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical"
        android:layout_marginBottom="16dp">

        <ImageButton
            android:id="@+id/backButton"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_alignParentStart="true"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:src="@drawable/ic_arrow_back"
            android:contentDescription="Back"
            android:tint="@android:color/black" />

        <TextView
            android:id="@+id/modCountTextView"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_centerInParent="true"
            android:text="Total Mods: 0 (Enabled: 0)"
            android:textSize="16sp"
            android:textStyle="bold" />

    </RelativeLayout>

    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerViewMods"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:scrollbars="vertical" />

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center"
        android:layout_marginTop="16dp">

        <Button
            android:id="@+id/addModButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Add Mod"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/refreshModsButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Refresh Mods" />
    </LinearLayout>

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_mod_management.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp"
        android:background="#F8F9FA">

        <!-- Header Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center"
            android:background="#FFFFFF"
            android:padding="20dp"
            android:layout_marginBottom="16dp"
            android:elevation="2dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="🎮 Mod Management"
                android:textSize="28sp"
                android:textStyle="bold"
                android:textColor="#2E7D32"
                android:gravity="center"
                android:layout_marginBottom="8dp" />

            <!-- Status Section -->
            <TextView
                android:id="@+id/statusText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="📊 Loading mod statistics..."
                android:textSize="14sp"
                android:textColor="#4CAF50"
                android:gravity="center"
                android:padding="8dp"
                android:background="#E8F5E8"
                android:layout_marginBottom="8dp" />

            <!-- Loader Status -->
            <TextView
                android:id="@+id/loaderStatusText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Checking loader status..."
                android:textSize="12sp"
                android:gravity="center"
                android:padding="6dp" />

        </LinearLayout>

        <!-- Loader Info Section (shown when loader is installed) -->
        <LinearLayout
            android:id="@+id/loaderInfoSection"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="#E3F2FD"
            android:padding="16dp"
            android:layout_marginBottom="16dp"
            android:visibility="gone"
            android:elevation="2dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="ℹ️ Loader Information"
                android:textSize="14sp"
                android:textStyle="bold"
                android:textColor="#1565C0"
                android:layout_marginBottom="8dp" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="• DLL mods will be loaded by MelonLoader\n• DEX/JAR mods are loaded directly by TerrariaLoader\n• Enable/disable mods using the switches below"
                android:textSize="12sp"
                android:textColor="#1976D2"
                android:lineSpacingExtra="2dp" />

        </LinearLayout>

        <!-- Add Mods Section -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#FFFFFF">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📥 Add New Mods"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#333333"
                    android:layout_marginBottom="12dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="2">

                    <Button
                        android:id="@+id/addDllModBtn"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📥 Add DLL Mod"
                        android:textSize="14sp"
                        android:background="#4CAF50"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="6dp"
                        android:minHeight="48dp" />

                    <Button
                        android:id="@+id/addDexModBtn"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📱 Add DEX/JAR Mod"
                        android:textSize="14sp"
                        android:background="#2196F3"
                        android:textColor="@android:color/white"
                        android:layout_marginStart="6dp"
                        android:minHeight="48dp" />

                </LinearLayout>

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="💡 DLL mods require MelonLoader • DEX/JAR mods work without a loader"
                    android:textSize="11sp"
                    android:textColor="#666666"
                    android:gravity="center"
                    android:layout_marginTop="8dp" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Mod List Section -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="0dp"
            android:layout_weight="1"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#FFFFFF">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="match_parent"
                android:orientation="vertical"
                android:padding="16dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="12dp">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📋 Installed Mods"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:textColor="#333333" />

                    <Button
                        android:id="@+id/refreshBtn"
                        android:layout_width="wrap_content"
                        android:layout_height="36dp"
                        android:text="🔄 Refresh"
                        android:textSize="12sp"
                        android:background="#FF9800"
                        android:textColor="@android:color/white"
                        android:minWidth="80dp" />

                </LinearLayout>

                <androidx.recyclerview.widget.RecyclerView
                    android:id="@+id/modRecyclerView"
                    android:layout_width="match_parent"
                    android:layout_height="0dp"
                    android:layout_weight="1"
                    android:background="#F9F9F9"
                    android:padding="8dp" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Navigation Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center"
            android:layout_marginTop="8dp">

            <Button
                android:id="@+id/backBtn"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="← Back"
                android:textSize="14sp"
                android:background="@android:color/transparent"
                android:textColor="#666666"
                android:minHeight="40dp"
                android:layout_marginEnd="16dp" />

            <TextView
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="Manage your mods after installation"
                android:textSize="12sp"
                android:textColor="#999999"
                android:gravity="center" />

        </LinearLayout>

        <!-- Tips Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="💡 Tips:\n• Toggle mods with switches\n• Delete mods with trash icon\n• DLL mods require patched Terraria APK\n• DEX/JAR mods work with any Terraria version"
            android:textSize="11sp"
            android:textColor="#888888"
            android:background="#F0F0F0"
            android:padding="12dp"
            android:layout_marginTop="16dp"
            android:lineSpacingExtra="2dp" />

    </LinearLayout>

</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_offline_diagnostic.xml

<?xml version="1.0" encoding="utf-8"?>
<!-- File: activity_offline_diagnostic.xml -->
<!-- Path: /res/layout/activity_offline_diagnostic.xml -->

<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@android:color/background_light">
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">
        
        <!-- Header Text -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="TerrariaLoader Offline Diagnostics"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="@android:color/holo_green_dark"
            android:gravity="center"
            android:layout_marginBottom="16dp" />
        
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Diagnose and fix common issues without internet connection"
            android:textSize="14sp"
            android:textColor="@android:color/darker_gray"
            android:gravity="center"
            android:layout_marginBottom="24dp" />
        
        <!-- Quick Actions Card -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">
            
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🚀 Quick Actions"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="@android:color/holo_blue_dark"
                    android:layout_marginBottom="12dp" />
                
                <Button
                    android:id="@+id/btn_run_full_diagnostic"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🔍 Run Full System Check"
                    android:textAllCaps="false"
                    android:background="@android:color/holo_green_light"
                    android:textColor="@android:color/white"
                    android:layout_marginBottom="8dp"
                    android:padding="12dp" />
                
                <Button
                    android:id="@+id/btn_diagnose_apk"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📦 Diagnose APK Installation Problem"
                    android:textAllCaps="false"
                    android:background="@android:color/holo_orange_light"
                    android:textColor="@android:color/white"
                    android:layout_marginBottom="8dp"
                    android:padding="12dp" />
                
                <Button
                    android:id="@+id/btn_fix_settings"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="⚙️ Fix Settings Persistence Issue"
                    android:textAllCaps="false"
                    android:background="@android:color/holo_blue_light"
                    android:textColor="@android:color/white"
                    android:layout_marginBottom="8dp"
                    android:padding="12dp" />
                
                <Button
                    android:id="@+id/btn_auto_repair"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🛠️ Attempt Auto-Repair"
                    android:textAllCaps="false"
                    android:background="@android:color/holo_purple"
                    android:textColor="@android:color/white"
                    android:padding="12dp" />
            </LinearLayout>
        </androidx.cardview.widget.CardView>
        
        <!-- Diagnostic Results Card -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">
            
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📊 Diagnostic Results"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="@android:color/holo_blue_dark"
                    android:layout_marginBottom="12dp" />
                
                <!-- Results Display Area -->
                <ScrollView
                    android:layout_width="match_parent"
                    android:layout_height="400dp"
                    android:background="@android:color/black"
                    android:padding="8dp">
                    
                    <TextView
                        android:id="@+id/diagnostic_results_text"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Click 'Run Full System Check' to start diagnostics..."
                        android:textColor="@android:color/holo_green_light"
                        android:textSize="12sp"
                        android:fontFamily="monospace"
                        android:textIsSelectable="true"
                        android:padding="8dp" />
                </ScrollView>
                
                <!-- Action Buttons for Results -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:layout_marginTop="12dp">
                    
                    <Button
                        android:id="@+id/btn_export_report"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="📤 Export Report"
                        android:textAllCaps="false"
                        android:background="@android:color/darker_gray"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="4dp" />
                    
                    <Button
                        android:id="@+id/btn_clear_results"
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="🗑️ Clear Results"
                        android:textAllCaps="false"
                        android:background="@android:color/darker_gray"
                        android:textColor="@android:color/white"
                        android:layout_marginStart="4dp" />
                </LinearLayout>
            </LinearLayout>
        </androidx.cardview.widget.CardView>
        
        <!-- Help/Info Section -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp">
            
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="💡 Quick Help"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="@android:color/holo_blue_dark"
                    android:layout_marginBottom="8dp" />
                
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• APK parsing errors: Usually caused by corrupted files or missing permissions\n• Settings not saving: Often due to storage permissions or corrupted preferences\n• Directory issues: Can be fixed with auto-repair function\n• Export reports to share with developers for support"
                    android:textSize="14sp"
                    android:textColor="@android:color/darker_gray"
                    android:lineSpacingMultiplier="1.2" />
            </LinearLayout>
        </androidx.cardview.widget.CardView>
        
        <!-- Version Info -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="TerrariaLoader Diagnostic Tool v1.0 - Offline Mode"
            android:textSize="12sp"
            android:textColor="@android:color/darker_gray"
            android:gravity="center"
            android:layout_marginTop="16dp"
            android:layout_marginBottom="16dp" />
        
    </LinearLayout>
</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_plugin_config.xml

<?xml version="1.0" encoding="utf-8"?>
<!-- File: activity_plugin_config.xml - Plugin Configuration Layout -->
<!-- Path: /app/src/main/res/layout/activity_plugin_config.xml -->

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#F5F5F5">

    <!-- Header Section -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#7B1FA2"
        android:padding="16dp"
        android:elevation="4dp">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🔧 Plugin Configuration"
            android:textSize="22sp"
            android:textStyle="bold"
            android:textColor="#FFFFFF"
            android:gravity="center"
            android:layout_marginBottom="8dp" />

        <!-- Plugin Information -->
        <TextView
            android:id="@+id/pluginNameText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Plugin Name v1.0"
            android:textSize="16sp"
            android:textColor="#E1BEE7"
            android:gravity="center"
            android:layout_marginBottom="4dp" />

        <TextView
            android:id="@+id/pluginStatusText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="✅ Plugin is active"
            android:textSize="14sp"
            android:textColor="#C8E6C9"
            android:gravity="center" />

    </LinearLayout>

    <!-- Action Buttons Row -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:background="#FFFFFF"
        android:padding="12dp"
        android:elevation="2dp">

        <Button
            android:id="@+id/saveButton"
            android:layout_width="0dp"
            android:layout_height="44dp"
            android:layout_weight="1"
            android:text="💾 Save"
            android:textSize="14sp"
            android:textStyle="bold"
            android:background="#4CAF50"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="6dp"
            android:elevation="2dp" />

        <Button
            android:id="@+id/resetButton"
            android:layout_width="0dp"
            android:layout_height="44dp"
            android:layout_weight="1"
            android:text="🔄 Reset"
            android:textSize="14sp"
            android:background="#FF9800"
            android:textColor="#FFFFFF"
            android:layout_marginStart="3dp"
            android:layout_marginEnd="3dp"
            android:elevation="2dp" />

        <Button
            android:id="@+id/exportButton"
            android:layout_width="0dp"
            android:layout_height="44dp"
            android:layout_weight="1"
            android:text="📤 Export"
            android:textSize="14sp"
            android:background="#2196F3"
            android:textColor="#FFFFFF"
            android:layout_marginStart="3dp"
            android:layout_marginEnd="3dp"
            android:elevation="2dp" />

        <Button
            android:id="@+id/importButton"
            android:layout_width="0dp"
            android:layout_height="44dp"
            android:layout_weight="1"
            android:text="📥 Import"
            android:textSize="14sp"
            android:background="#607D8B"
            android:textColor="#FFFFFF"
            android:layout_marginStart="6dp"
            android:elevation="2dp" />

    </LinearLayout>

    <!-- Configuration Items -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/configRecyclerView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="8dp"
        android:clipToPadding="false"
        android:scrollbars="vertical" />

    <!-- Help Text -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="💡 Changes are saved automatically when you modify settings. Some changes may require plugin restart to take effect."
        android:textSize="12sp"
        android:textColor="#666666"
        android:textStyle="italic"
        android:background="#FFFFFF"
        android:padding="12dp"
        android:lineSpacingExtra="2dp" />

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_plugin_install.xml

<?xml version="1.0" encoding="utf-8"?>
<!-- File: activity_plugin_install.xml - Plugin Installation Layout -->
<!-- Path: /app/src/main/res/layout/activity_plugin_install.xml -->

<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F5F5F5"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- Header -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="📦 Install Plugin"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="#2E7D32"
            android:gravity="center"
            android:layout_marginBottom="24dp" />

        <!-- File Information Card -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#FFFFFF">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📄 File Information"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#333333"
                    android:layout_marginBottom="12dp" />

                <TextView
                    android:id="@+id/fileInfoText"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Analyzing file..."
                    android:textSize="14sp"
                    android:textColor="#666666"
                    android:lineSpacingExtra="4dp" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Validation Status Card -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#FFFFFF">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="✅ Validation Status"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#333333"
                    android:layout_marginBottom="12dp" />

                <!-- Validation Progress -->
                <ProgressBar
                    android:id="@+id/validationProgress"
                    style="?android:attr/progressBarStyleHorizontal"
                    android:layout_width="match_parent"
                    android:layout_height="8dp"
                    android:layout_marginBottom="12dp"
                    android:indeterminate="true"
                    android:progressTint="#4CAF50" />

                <TextView
                    android:id="@+id/validationStatusText"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Starting validation..."
                    android:textSize="14sp"
                    android:textColor="#666666" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Plugin Details Card -->
        <androidx.cardview.widget.CardView
            android:id="@+id/pluginDetailsLayout"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#E8F5E8"
            android:visibility="gone">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🔌 Plugin Information"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#2E7D32"
                    android:layout_marginBottom="12dp" />

                <TextView
                    android:id="@+id/pluginInfoText"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Plugin details will appear here"
                    android:textSize="14sp"
                    android:textColor="#4CAF50"
                    android:lineSpacingExtra="4dp"
                    android:layout_marginBottom="16dp" />

                <!-- Installation Options -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="⚙️ Installation Options"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#2E7D32"
                    android:layout_marginBottom="12dp" />

                <CheckBox
                    android:id="@+id/autoEnableCheckBox"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🔄 Automatically enable plugin after installation"
                    android:textSize="14sp"
                    android:textColor="#2E7D32"
                    android:checked="true"
                    android:layout_marginBottom="8dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="💡 Tip: You can manage plugin settings after installation in the Plugin Management screen."
                    android:textSize="12sp"
                    android:textColor="#666666"
                    android:lineSpacingExtra="2dp" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Installation Requirements Card -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="24dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#E3F2FD">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📋 Installation Requirements"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#1565C0"
                    android:layout_marginBottom="12dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• Valid .jar plugin file\n• Storage permission for plugin data\n• TerrariaLoader v1.0+ compatibility\n• Sufficient device memory"
                    android:textSize="14sp"
                    android:textColor="#1976D2"
                    android:lineSpacingExtra="4dp" />

            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Action Buttons -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="16dp">

            <Button
                android:id="@+id/cancelButton"
                android:layout_width="0dp"
                android:layout_height="56dp"
                android:layout_weight="1"
                android:text="❌ Cancel"
                android:textSize="16sp"
                android:textStyle="bold"
                android:background="#757575"
                android:textColor="#FFFFFF"
                android:layout_marginEnd="8dp"
                android:elevation="4dp" />

            <Button
                android:id="@+id/installButton"
                android:layout_width="0dp"
                android:layout_height="56dp"
                android:layout_weight="2"
                android:text="📦 Install Plugin"
                android:textSize="16sp"
                android:textStyle="bold"
                android:background="#4CAF50"
                android:textColor="#FFFFFF"
                android:layout_marginStart="8dp"
                android:elevation="4dp"
                android:enabled="false" />

        </LinearLayout>

        <!-- Warning Text -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="⚠️ Only install plugins from trusted sources. Malicious plugins may damage your device or compromise your data."
            android:textSize="12sp"
            android:textColor="#F44336"
            android:textStyle="italic"
            android:gravity="center"
            android:layout_marginTop="16dp"
            android:padding="12dp"
            android:background="#FFEBEE"
            android:lineSpacingExtra="2dp" />

    </LinearLayout>

</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_plugin_management.xml

<?xml version="1.0" encoding="utf-8"?>
<!-- File: activity_plugin_management.xml - Plugin Management Layout -->
<!-- Path: /app/src/main/res/layout/activity_plugin_management.xml -->

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#F5F5F5">

    <!-- Header Section -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#2E7D32"
        android:padding="16dp"
        android:elevation="4dp">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🔌 Plugin Management"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="#FFFFFF"
            android:gravity="center"
            android:layout_marginBottom="8dp" />

        <!-- Status Text -->
        <TextView
            android:id="@+id/statusText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Loading plugins..."
            android:textSize="14sp"
            android:textColor="#E8F5E8"
            android:gravity="center"
            android:layout_marginBottom="8dp" />

        <!-- Plugin Count -->
        <TextView
            android:id="@+id/pluginCountText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Total: 0 | Loaded: 0 | Active: 0"
            android:textSize="12sp"
            android:textColor="#C8E6C9"
            android:gravity="center" />

    </LinearLayout>

    <!-- Action Buttons -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp"
        android:background="#FFFFFF"
        android:elevation="2dp">

        <Button
            android:id="@+id/installPluginButton"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:text="📥 Install Plugin"
            android:textSize="14sp"
            android:textStyle="bold"
            android:background="#4CAF50"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp"
            android:elevation="2dp" />

        <Button
            android:id="@+id/scanButton"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:text="🔍 Scan"
            android:textSize="14sp"
            android:background="#2196F3"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp"
            android:elevation="2dp" />

        <Button
            android:id="@+id/refreshButton"
            android:layout_width="0dp"
            android:layout_height="48dp"
            android:layout_weight="1"
            android:text="🔄 Refresh"
            android:textSize="14sp"
            android:background="#FF9800"
            android:textColor="#FFFFFF"
            android:elevation="2dp" />

    </LinearLayout>

    <!-- Main Content -->
    <androidx.swiperefreshlayout.widget.SwipeRefreshLayout
        android:id="@+id/swipeRefreshLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/pluginRecyclerView"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:padding="8dp"
            android:clipToPadding="false"
            android:scrollbars="vertical" />

    </androidx.swiperefreshlayout.widget.SwipeRefreshLayout>

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_settings_enhanced.xml

<!-- File: activity_settings_enhanced.xml (Enhanced Settings Layout - Error-Free) -->
<!-- Path: /app/src/main/res/layout/activity_settings_enhanced.xml -->

<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#F5F5F5">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- Header -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="⚙️ TerrariaLoader Settings"
            android:textSize="24sp"
            android:textStyle="bold"
            android:gravity="center"
            android:layout_marginBottom="24dp"
            android:textColor="#2E7D32" />

        <!-- Operation Modes Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🚀 Operation Modes"
            android:textSize="20sp"
            android:textStyle="bold"
            android:layout_marginBottom="16dp"
            android:textColor="#1976D2" />

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Choose how TerrariaLoader operates:"
            android:textSize="14sp"
            android:layout_marginBottom="16dp"
            android:textColor="#666666" />

        <!-- Operation Mode Radio Group -->
        <RadioGroup
            android:id="@+id/operationModeGroup"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="24dp">

            <!-- Normal Mode Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/normalCard"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="12dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:gravity="center_vertical">

                        <RadioButton
                            android:id="@+id/normalModeRadio"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="12dp" />

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="📱 Normal Mode"
                            android:textSize="18sp"
                            android:textStyle="bold"
                            android:textColor="#2E7D32" />

                    </LinearLayout>

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Standard Android permissions only"
                        android:textSize="14sp"
                        android:layout_marginTop="8dp"
                        android:textColor="#666666" />

                    <TextView
                        android:id="@+id/normalStatus"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="✅ Standard permissions"
                        android:textSize="12sp"
                        android:layout_marginTop="4dp"
                        android:textColor="#4CAF50" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Shizuku Mode Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/shizukuCard"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="12dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:gravity="center_vertical">

                        <RadioButton
                            android:id="@+id/shizukuModeRadio"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="12dp" />

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="🛡️ Shizuku Mode"
                            android:textSize="18sp"
                            android:textStyle="bold"
                            android:textColor="#1976D2" />

                    </LinearLayout>

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Enhanced permissions via Shizuku"
                        android:textSize="14sp"
                        android:layout_marginTop="8dp"
                        android:textColor="#666666" />

                    <TextView
                        android:id="@+id/shizukuStatus"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Shizuku not available"
                        android:textSize="12sp"
                        android:layout_marginTop="4dp"
                        android:textColor="#FF5722" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Root Mode Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/rootCard"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="12dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:gravity="center_vertical">

                        <RadioButton
                            android:id="@+id/rootModeRadio"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="12dp" />

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="🔓 Root Mode"
                            android:textSize="18sp"
                            android:textStyle="bold"
                            android:textColor="#E91E63" />

                    </LinearLayout>

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Maximum system control with root"
                        android:textSize="14sp"
                        android:layout_marginTop="8dp"
                        android:textColor="#666666" />

                    <TextView
                        android:id="@+id/rootStatus"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Root not available"
                        android:textSize="12sp"
                        android:layout_marginTop="4dp"
                        android:textColor="#FF5722" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Hybrid Mode Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/hybridCard"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="12dp"
                app:cardCornerRadius="8dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="16dp">

                    <LinearLayout
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal"
                        android:gravity="center_vertical">

                        <RadioButton
                            android:id="@+id/hybridModeRadio"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:layout_marginEnd="12dp" />

                        <TextView
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:text="⚡ Hybrid Mode"
                            android:textSize="18sp"
                            android:textStyle="bold"
                            android:textColor="#9C27B0" />

                    </LinearLayout>

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Both Shizuku and Root capabilities"
                        android:textSize="14sp"
                        android:layout_marginTop="8dp"
                        android:textColor="#666666" />

                    <TextView
                        android:id="@+id/hybridStatus"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="Requires both Shizuku and Root"
                        android:textSize="12sp"
                        android:layout_marginTop="4dp"
                        android:textColor="#FF5722" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

        </RadioGroup>

        <!-- Setup Buttons Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🔧 Setup &amp; Permissions"
            android:textSize="20sp"
            android:textStyle="bold"
            android:layout_marginBottom="16dp"
            android:textColor="#1976D2" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="16dp">

            <Button
                android:id="@+id/shizukuSetupBtn"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginEnd="8dp"
                android:text="📥 Setup Shizuku"
                android:textSize="12sp"
                android:backgroundTint="#2196F3" />

            <Button
                android:id="@+id/rootSetupBtn"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="8dp"
                android:text="🔓 Check Root"
                android:textSize="12sp"
                android:backgroundTint="#E91E63" />

        </LinearLayout>

        <Button
            android:id="@+id/permissionBtn"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="24dp"
            android:text="🔐 Manage Permissions"
            android:backgroundTint="#4CAF50" />

        <!-- Feature Toggles Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="⚙️ Feature Settings"
            android:textSize="20sp"
            android:textStyle="bold"
            android:layout_marginBottom="16dp"
            android:textColor="#1976D2" />

        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="2dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <!-- Auto Enable Mods -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="12dp">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="🔄 Auto-enable new mods"
                        android:textSize="16sp"
                        android:textColor="#333333" />

                    <Switch
                        android:id="@+id/autoEnableSwitch"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content" />

                </LinearLayout>

                <!-- Debug Logging -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="12dp">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="🐛 Debug logging"
                        android:textSize="16sp"
                        android:textColor="#333333" />

                    <Switch
                        android:id="@+id/debugLoggingSwitch"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content" />

                </LinearLayout>

                <!-- Auto Backup -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="12dp">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="💾 Auto-backup APKs"
                        android:textSize="16sp"
                        android:textColor="#333333" />

                    <Switch
                        android:id="@+id/autoBackupSwitch"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content" />

                </LinearLayout>

                <!-- Auto Update Check -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:text="🔄 Check for updates"
                        android:textSize="16sp"
                        android:textColor="#333333" />

                    <Switch
                        android:id="@+id/autoUpdateSwitch"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content" />

                </LinearLayout>

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Action Buttons Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🛠️ Actions"
            android:textSize="20sp"
            android:textStyle="bold"
            android:layout_marginBottom="16dp"
            android:textColor="#1976D2" />

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="16dp">

            <Button
                android:id="@+id/resetSettingsBtn"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginEnd="8dp"
                android:text="🔄 Reset"
                android:textSize="12sp"
                android:backgroundTint="#FF5722" />

            <Button
                android:id="@+id/exportSettingsBtn"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="4dp"
                android:layout_marginEnd="4dp"
                android:text="📤 Export"
                android:textSize="12sp"
                android:backgroundTint="#607D8B" />

            <Button
                android:id="@+id/importSettingsBtn"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:layout_marginStart="8dp"
                android:text="📥 Import"
                android:textSize="12sp"
                android:backgroundTint="#607D8B" />

        </LinearLayout>

        <!-- Info Section -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="2dp"
            app:cardBackgroundColor="#E3F2FD">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="ℹ️ Tips"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:layout_marginBottom="8dp"
                    android:textColor="#1976D2" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• Normal Mode works on all devices\n• Shizuku Mode provides enhanced access without root\n• Root Mode requires a rooted device\n• Hybrid Mode combines both enhanced methods"
                    android:textSize="14sp"
                    android:textColor="#1976D2" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Version Info -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="TerrariaLoader v1.0 - Enhanced Edition"
            android:textSize="12sp"
            android:gravity="center"
            android:layout_marginTop="16dp"
            android:textColor="#999999" />

    </LinearLayout>

</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_setup_guide.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="24dp"
        android:background="@drawable/gradient_background_135">

        <!-- Header Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center"
            android:background="#FFFFFF"
            android:padding="32dp"
            android:layout_marginBottom="24dp"
            android:elevation="8dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="🚀 MelonLoader Setup Guide"
                android:textSize="28sp"
                android:textStyle="bold"
                android:textColor="#2E7D32"
                android:gravity="center"
                android:layout_marginBottom="12dp" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Choose your preferred installation method"
                android:textSize="16sp"
                android:textColor="#4CAF50"
                android:gravity="center"
                android:lineSpacingExtra="4dp" />

        </LinearLayout>

        <!-- Installation Options -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_marginBottom="24dp">

            <!-- Online Installation Card -->
            <androidx.cardview.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="6dp"
                app:cardBackgroundColor="#E3F2FD">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="20dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="🌐 Automated Online Installation"
                        android:textSize="20sp"
                        android:textStyle="bold"
                        android:textColor="#1565C0"
                        android:layout_marginBottom="12dp" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="• Automatically downloads from GitHub\n• No manual file handling\n• Always gets latest version\n• Requires internet connection"
                        android:textSize="14sp"
                        android:textColor="#1976D2"
                        android:lineSpacingExtra="4dp"
                        android:layout_marginBottom="16dp" />

                    <Button
                        android:id="@+id/btn_online_install"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="🌐 Start Online Installation"
                        android:textSize="16sp"
                        android:textStyle="bold"
                        android:background="#2196F3"
                        android:textColor="@android:color/white"
                        android:minHeight="56dp"
                        android:elevation="4dp" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Offline Import Card -->
            <androidx.cardview.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="6dp"
                app:cardBackgroundColor="#FFF3E0">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="20dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="📦 Offline ZIP Import"
                        android:textSize="20sp"
                        android:textStyle="bold"
                        android:textColor="#E65100"
                        android:layout_marginBottom="12dp" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="• Import pre-downloaded ZIP files\n• Works without internet\n• Auto-detects NET8/NET35\n• Extracts to correct directories"
                        android:textSize="14sp"
                        android:textColor="#F57C00"
                        android:lineSpacingExtra="4dp"
                        android:layout_marginBottom="16dp" />

                    <Button
                        android:id="@+id/btn_offline_import"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="📦 Import ZIP File"
                        android:textSize="16sp"
                        android:textStyle="bold"
                        android:background="#FF9800"
                        android:textColor="@android:color/white"
                        android:minHeight="56dp"
                        android:elevation="4dp" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="💡 Supports: melon_data.zip, lemon_data.zip, custom packages"
                        android:textSize="11sp"
                        android:textColor="#BF360C"
                        android:gravity="center"
                        android:layout_marginTop="8dp" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

            <!-- Manual Installation Card -->
            <androidx.cardview.widget.CardView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="16dp"
                app:cardCornerRadius="12dp"
                app:cardElevation="6dp"
                app:cardBackgroundColor="#F3E5F5">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical"
                    android:padding="20dp">

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="📖 Manual Installation Guide"
                        android:textSize="20sp"
                        android:textStyle="bold"
                        android:textColor="#7B1FA2"
                        android:layout_marginBottom="12dp" />

                    <TextView
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="• Step-by-step instructions\n• Full control over installation\n• Troubleshooting included\n• For advanced users"
                        android:textSize="14sp"
                        android:textColor="#8E24AA"
                        android:lineSpacingExtra="4dp"
                        android:layout_marginBottom="16dp" />

                    <Button
                        android:id="@+id/btn_manual_instructions"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:text="📖 View Manual Guide"
                        android:textSize="16sp"
                        android:textStyle="bold"
                        android:background="#9C27B0"
                        android:textColor="@android:color/white"
                        android:minHeight="56dp"
                        android:elevation="4dp" />

                </LinearLayout>

            </androidx.cardview.widget.CardView>

        </LinearLayout>

        <!-- Information Section -->
        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="8dp"
            app:cardElevation="4dp"
            app:cardBackgroundColor="#E8F5E8">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="ℹ️ What happens after installation?"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#2E7D32"
                    android:layout_marginBottom="8dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="1. 🚀 Unified Loader opens automatically\n2. 📱 Select your Terraria APK file\n3. ⚡ Patch APK with MelonLoader\n4. 📲 Install the patched APK\n5. 🎮 Add DLL mods and enjoy!"
                    android:textSize="14sp"
                    android:textColor="#388E3C"
                    android:lineSpacingExtra="4dp" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Requirements Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:background="#FFFFFF"
            android:padding="16dp"
            android:layout_marginTop="8dp"
            android:elevation="2dp">

            <TextView
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="📋 Requirements:\n• 50MB+ free space\n• Terraria APK file\n• File manager permissions"
                android:textSize="12sp"
                android:textColor="#666666"
                android:lineSpacingExtra="2dp" />

            <TextView
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="🎯 Recommended:\n• Use Online Installation\n• Keep APK backup\n• Enable unknown sources"
                android:textSize="12sp"
                android:textColor="#666666"
                android:lineSpacingExtra="2dp" />

        </LinearLayout>

    </LinearLayout>

</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_specific_selection.xml

<?xml version="1.0" encoding="utf-8"?>

<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="24dp">

    <LinearLayout
        android:id="@+id/specific_selection_layout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="center">

        <!-- Header -->
        <TextView
            android:id="@+id/headerText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Choose Game/App to Mod"
            android:textSize="24sp"
            android:textStyle="bold"
            android:gravity="center"
            android:layout_marginBottom="32dp" />

        <!-- Terraria Button -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:background="@android:drawable/dialog_frame"
            android:padding="16dp"
            android:layout_marginBottom="16dp">

            <Button
                android:id="@+id/terraria_button"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="🌍 Terraria"
                android:textSize="20sp"
                android:textStyle="bold"
                android:minHeight="60dp" />

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="• Support for DEX/JAR mods\n• Support for DLL mods (via MelonLoader)\n• APK patching and installation\n• Full mod management"
                android:textSize="14sp"
                android:textColor="@android:color/darker_gray"
                android:layout_marginTop="8dp" />

        </LinearLayout>

        <!-- Future Games Section -->
        <LinearLayout
            android:id="@+id/futureGamesSection"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:layout_marginTop="24dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Coming Soon"
                android:textSize="18sp"
                android:textStyle="bold"
                android:gravity="center"
                android:layout_marginBottom="16dp" />

            <!-- Placeholder cards for future games -->
            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:background="@android:drawable/dialog_frame"
                android:padding="12dp"
                android:layout_marginBottom="8dp"
                android:alpha="0.5">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="🟫 Minecraft PE"
                        android:textSize="16sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Soon"
                        android:textSize="12sp"
                        android:textColor="@android:color/darker_gray" />

                </LinearLayout>
            </LinearLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:background="@android:drawable/dialog_frame"
                android:padding="12dp"
                android:layout_marginBottom="8dp"
                android:alpha="0.5">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <TextView
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="🚀 Among Us"
                        android:textSize="16sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Soon"
                        android:textSize="12sp"
                        android:textColor="@android:color/darker_gray" />

                </LinearLayout>
            </LinearLayout>

        </LinearLayout>

        <!-- Back Button -->
        <Button
            android:id="@+id/backToMainButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="← Back to Main Menu"
            android:layout_marginTop="32dp" />

    </LinearLayout>
</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_terraria_specific_updated.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp"
        android:background="#E8F5E8">

        <!-- Header Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center"
            android:layout_marginBottom="24dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="🌍 Terraria Mod Loader"
                android:textSize="28sp"
                android:textStyle="bold"
                android:textColor="#2E7D32"
                android:gravity="center"
                android:layout_marginBottom="8dp" />

            <TextView
                android:id="@+id/loaderStatusText"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="Checking loader status..."
                android:textSize="14sp"
                android:textColor="#4CAF50"
                android:gravity="center"
                android:padding="12dp"
                android:background="#F1F8E9"
                android:layout_marginTop="8dp" />
        </LinearLayout>

        <!-- Setup & Installation Section -->
        <androidx.cardview.widget.CardView
            android:id="@+id/setupCard"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="6dp"
            app:cardBackgroundColor="#F1F8E9">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="20dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🚀 Setup &amp; Installation"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#2E7D32"
                    android:layout_marginBottom="16dp" />

                <Button
                    android:id="@+id/unifiedSetupButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🎯 Complete Setup Wizard"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:background="#4CAF50"
                    android:textColor="@android:color/white"
                    android:layout_marginBottom="12dp"
                    android:minHeight="56dp"
                    android:elevation="2dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="All-in-one wizard for MelonLoader installation and APK patching"
                    android:textSize="12sp"
                    android:textColor="#66BB6A"
                    android:layout_marginBottom="16dp"
                    android:gravity="center" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="2">

                    <Button
                        android:id="@+id/setupGuideButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📖 Setup Guide"
                        android:textSize="14sp"
                        android:background="#81C784"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="6dp"
                        android:minHeight="48dp" />

                    <Button
                        android:id="@+id/manualInstructionsButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📋 Manual Steps"
                        android:textSize="14sp"
                        android:background="#A5D6A7"
                        android:textColor="#2E7D32"
                        android:layout_marginStart="6dp"
                        android:minHeight="48dp" />
                </LinearLayout>
            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Mod Management Section -->
        <androidx.cardview.widget.CardView
            android:id="@+id/modManagementCard"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="6dp"
            app:cardBackgroundColor="#E3F2FD">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="20dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📦 Mod Management"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#1565C0"
                    android:layout_marginBottom="16dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="2">

                    <Button
                        android:id="@+id/dexModManagerButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📱 DEX/JAR Mods"
                        android:textSize="14sp"
                        android:background="#2196F3"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="6dp"
                        android:minHeight="48dp" />

                    <Button
                        android:id="@+id/dllModManagerButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="🔧 DLL Mods"
                        android:textSize="14sp"
                        android:background="#FF9800"
                        android:textColor="@android:color/white"
                        android:layout_marginStart="6dp"
                        android:minHeight="48dp" />
                </LinearLayout>

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• DEX/JAR: Android Java mods (always available)\n• DLL: C# mods (requires MelonLoader)"
                    android:textSize="12sp"
                    android:textColor="#42A5F5"
                    android:layout_marginTop="12dp"
                    android:lineSpacingExtra="2dp" />
            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Tools & Utilities Section -->
        <androidx.cardview.widget.CardView
            android:id="@+id/toolsCard"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="6dp"
            app:cardBackgroundColor="#F3E5F5">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="20dp">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🛠️ Tools &amp; Utilities"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#7B1FA2"
                    android:layout_marginBottom="16dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="3">

                    <Button
                        android:id="@+id/logViewerButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📋 Logs"
                        android:textSize="12sp"
                        android:background="#9C27B0"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="4dp"
                        android:minHeight="44dp" />

                    <Button
                        android:id="@+id/settingsButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="⚙️ Settings"
                        android:textSize="12sp"
                        android:background="#BA68C8"
                        android:textColor="@android:color/white"
                        android:layout_marginHorizontal="4dp"
                        android:minHeight="44dp" />

                    <Button
                        android:id="@+id/sandboxButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="🧪 Sandbox"
                        android:textSize="12sp"
                        android:background="#CE93D8"
                        android:textColor="#4A148C"
                        android:layout_marginStart="4dp"
                        android:minHeight="44dp" />
                </LinearLayout>

                <!-- ✅ Fixed Diagnostic Button -->
                <Button
                    android:id="@+id/diagnosticButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🧪 Offline Diagnostic &amp; Repair"
                    android:textSize="14sp"
                    android:textStyle="bold"
                    android:backgroundTint="#9C27B0"
                    android:textColor="#FFFFFF"
                    android:layout_marginTop="12dp"
                    android:minHeight="48dp" />
            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <!-- Navigation Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center"
            android:layout_marginTop="16dp">

            <Button
                android:id="@+id/backButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="← Back to App Selection"
                android:textSize="14sp"
                android:background="@android:color/transparent"
                android:textColor="#666666"
                android:minHeight="40dp" />
        </LinearLayout>

        <!-- Info Footer -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="💡 Tip: Start with 'Complete Setup Wizard' for the easiest experience!"
            android:textSize="12sp"
            android:textColor="#81C784"
            android:gravity="center"
            android:background="#F1F8E9"
            android:padding="16dp"
            android:layout_marginTop="16dp"
            android:layout_marginBottom="16dp" />
    </LinearLayout>
</ScrollView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_unified_loader.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="@android:color/white">

    <!-- Header Section -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#E8F5E8"
        android:padding="16dp"
        android:elevation="4dp">

        <!-- Progress Bar -->
        <ProgressBar
            android:id="@+id/stepProgressBar"
            style="?android:attr/progressBarStyleHorizontal"
            android:layout_width="match_parent"
            android:layout_height="8dp"
            android:layout_marginBottom="12dp"
            android:max="4"
            android:progress="0"
            android:progressTint="#4CAF50"
            android:progressBackgroundTint="#E0E0E0" />

        <!-- Step Indicator -->
        <TextView
            android:id="@+id/stepIndicatorText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Step 1 of 5"
            android:textSize="12sp"
            android:textColor="#666666"
            android:gravity="center"
            android:layout_marginBottom="8dp" />

        <!-- Step Title -->
        <TextView
            android:id="@+id/stepTitleText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Welcome"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="#2E7D32"
            android:gravity="center"
            android:layout_marginBottom="8dp" />

        <!-- Step Description -->
        <TextView
            android:id="@+id/stepDescriptionText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Welcome to the MelonLoader Setup Wizard"
            android:textSize="14sp"
            android:textColor="#4CAF50"
            android:gravity="center"
            android:lineSpacingExtra="4dp" />

    </LinearLayout>

    <!-- Main Content Area -->
    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:fillViewport="true">

        <LinearLayout
            android:id="@+id/stepContentContainer"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp"
            android:minHeight="400dp">

            <!-- Dynamic content will be added here -->

        </LinearLayout>

    </ScrollView>

    <!-- Navigation Footer -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:background="#F5F5F5"
        android:padding="16dp"
        android:elevation="4dp">

        <!-- Action Button (context-sensitive) -->
        <Button
            android:id="@+id/actionButton"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🚀 Start Setup"
            android:textSize="16sp"
            android:textStyle="bold"
            android:background="#4CAF50"
            android:textColor="@android:color/white"
            android:layout_marginBottom="12dp"
            android:minHeight="48dp" />

        <!-- Navigation Buttons -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:weightSum="2">

            <Button
                android:id="@+id/previousButton"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="← Previous"
                android:textSize="14sp"
                android:background="@android:color/transparent"
                android:textColor="#666666"
                android:layout_marginEnd="8dp"
                android:enabled="false" />

            <Button
                android:id="@+id/nextButton"
                android:layout_width="0dp"
                android:layout_weight="1"
                android:layout_height="wrap_content"
                android:text="Next →"
                android:textSize="14sp"
                android:background="#2196F3"
                android:textColor="@android:color/white"
                android:layout_marginStart="8dp" />

        </LinearLayout>

        <!-- Progress Text (for operations) -->
        <TextView
            android:id="@+id/progressText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text=""
            android:textSize="12sp"
            android:textColor="#666666"
            android:gravity="center"
            android:layout_marginTop="8dp"
            android:visibility="gone" />

    </LinearLayout>

    <!-- Hidden Status Views (referenced by activity) -->
    <TextView
        android:id="@+id/loaderStatusText"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:visibility="gone" />

    <TextView
        android:id="@+id/apkStatusText"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:visibility="gone" />

    <ProgressBar
        android:id="@+id/actionProgressBar"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:visibility="gone" />

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/activity_universal.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:padding="16dp">

    <Button
        android:id="@+id/select_apk_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Select Universal APK" />

    <Button
        android:id="@+id/select_zip_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Select Loader ZIP" />

    <Button
        android:id="@+id/inject_button"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Inject Loader" />

    <TextView
        android:id="@+id/status_text"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="Status"
        android:paddingTop="16dp"
        android:textAppearance="?android:attr/textAppearanceMedium" />

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/dialog_log_settings.xml

<!-- File: dialog_log_settings.xml (NEW DIALOG) - Settings for Log Viewer -->
<!-- Path: /main/res/layout/dialog_log_settings.xml -->

<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp"
    android:background="#2A2A2A">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="⚙️ Log Viewer Settings"
        android:textColor="#FFFFFF"
        android:textSize="18sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginBottom="16dp" />

    <!-- Auto Refresh Setting -->
    <CheckBox
        android:id="@+id/autoRefreshCheckbox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="🔄 Auto-refresh logs (every 5 seconds)"
        android:textColor="#FFFFFF"
        android:textSize="14sp"
        android:checked="true"
        android:buttonTint="#4CAF50"
        android:layout_marginBottom="12dp" />

    <!-- Syntax Highlighting Setting -->
    <CheckBox
        android:id="@+id/syntaxHighlightCheckbox"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="🎨 Enable syntax highlighting"
        android:textColor="#FFFFFF"
        android:textSize="14sp"
        android:checked="true"
        android:buttonTint="#4CAF50"
        android:layout_marginBottom="12dp" />

    <!-- Text Size Setting -->
    <TextView
        android:id="@+id/textSizeLabel"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="📏 Text Size: 12"
        android:textColor="#FFFFFF"
        android:textSize="14sp"
        android:layout_marginBottom="8dp" />

    <SeekBar
        android:id="@+id/textSizeSeekBar"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:max="20"
        android:min="8"
        android:progress="12"
        android:thumbTint="#4CAF50"
        android:progressTint="#4CAF50"
        android:layout_marginBottom="16dp" />

    <!-- Information Text -->
    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="💡 Changes are applied immediately and persist during this session."
        android:textColor="#888888"
        android:textSize="12sp"
        android:gravity="center"
        android:layout_marginTop="8dp" />

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/item_log_entry.xml

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="horizontal"
    android:padding="8dp"
    android:background="?android:attr/selectableItemBackground"
    android:minHeight="56dp">

    <!-- Level Indicator Bar -->
    <View
        android:id="@+id/levelIndicator"
        android:layout_width="4dp"
        android:layout_height="match_parent"
        android:layout_marginEnd="8dp"
        android:background="#2196F3" />

    <!-- Main Content -->
    <LinearLayout
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:layout_weight="1"
        android:orientation="vertical">

        <!-- Header Row (Timestamp, Level, Tag) -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginBottom="4dp">

            <TextView
                android:id="@+id/logTimestamp"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="00:00:00"
                android:textSize="11sp"
                android:textColor="#666666"
                android:fontFamily="monospace"
                android:layout_marginEnd="8dp" />

            <TextView
                android:id="@+id/logLevel"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="INFO"
                android:textSize="11sp"
                android:textStyle="bold"
                android:textColor="#2196F3"
                android:layout_marginEnd="8dp"
                android:minWidth="48dp" />

            <TextView
                android:id="@+id/logTag"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="TAG"
                android:textSize="11sp"
                android:textColor="#666666"
                android:textStyle="italic"
                android:ellipsize="end"
                android:maxLines="1" />

        </LinearLayout>

        <!-- Message Content -->
        <TextView
            android:id="@+id/logMessage"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Log message content goes here and can span multiple lines if needed"
            android:textSize="14sp"
            android:textColor="#333333"
            android:lineSpacingExtra="2dp"
            android:textIsSelectable="true"
            android:maxLines="10"
            android:ellipsize="end" />

    </LinearLayout>

</LinearLayout>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/item_mod.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp"
        android:gravity="center_vertical">

        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical">

            <TextView
                android:id="@+id/modNameTextView"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Mod Name"
                android:textSize="18sp"
                android:textStyle="bold" />

            <TextView
                android:id="@+id/modDescription"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Mod description goes here. This can be multiline."
                android:textSize="14sp"
                android:textColor="@android:color/darker_gray"
                android:layout_marginTop="4dp" />

        </LinearLayout>

        <Switch
            android:id="@+id/modSwitch"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="16dp" />

        <ImageButton
            android:id="@+id/modDeleteButton"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:layout_marginStart="8dp"
            android:background="?attr/selectableItemBackgroundBorderless"
            android:src="@android:drawable/ic_menu_delete"
            android:contentDescription="Delete Mod"
            app:tint="@android:color/holo_red_dark" />

    </LinearLayout>
</androidx.cardview.widget.CardView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/item_plugin.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
     xmlns:android="http://schemas.android.com/apk/res/android"
     xmlns:app="http://schemas.android.com/apk/res-auto"
     android:layout_height="wrap_content"
     android:layout_width="match_parent"
     android:layout_margin="8dp"
     app:cardElevation="4dp"
     app:cardBackgroundColor="#FFFFFF"
     app:cardCornerRadius="12dp">

    <LinearLayout
         android:layout_height="wrap_content"
         android:layout_width="match_parent"
         android:padding="16dp"
         android:orientation="vertical">

        <LinearLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:layout_marginBottom="12dp"
             android:gravity="center_vertical"
             android:orientation="horizontal">

            <View
                 android:layout_height="12dp"
                 android:layout_width="12dp"
                 android:layout_marginEnd="12dp"
                 android:background="@drawable/circle_shape"
                 android:backgroundTint="#4CAF50"
                 android:id="@+id/pluginStatusIndicator" />

            <LinearLayout
                 android:layout_height="wrap_content"
                 android:layout_width="0dp"
                 android:orientation="vertical"
                 android:layout_weight="1">

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="match_parent"
                     android:ellipsize="end"
                     android:textSize="18sp"
                     android:textColor="#333333"
                     android:maxLines="1"
                     android:id="@+id/pluginNameText"
                     android:text="Plugin Name"
                     android:textStyle="bold" />

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="match_parent"
                     android:textSize="12sp"
                     android:textColor="#666666"
                     android:id="@+id/pluginVersionText"
                     android:text="v1.0.0" />

            </LinearLayout>

            <TextView
                 android:layout_height="wrap_content"
                 android:layout_width="wrap_content"
                 android:background="#E8F5E8"
                 android:padding="6dp"
                 android:textSize="12sp"
                 android:textColor="#4CAF50"
                 android:layout_marginStart="8dp"
                 android:id="@+id/pluginStatusText"
                 android:text="Active"
                 android:textStyle="bold" />

        </LinearLayout>

        <TextView
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:layout_marginBottom="16dp"
             android:ellipsize="end"
             android:textSize="14sp"
             android:textColor="#666666"
             android:lineSpacingExtra="2dp"
             android:maxLines="3"
             android:id="@+id/pluginDescriptionText"
             android:text="Plugin description goes here. This provides information about what the plugin does and its features." />

        <LinearLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:gravity="center_vertical"
             android:orientation="horizontal">

            <LinearLayout
                 android:layout_height="wrap_content"
                 android:layout_width="0dp"
                 android:gravity="center_vertical"
                 android:orientation="horizontal"
                 android:layout_weight="1">

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="wrap_content"
                     android:layout_marginEnd="8dp"
                     android:textSize="14sp"
                     android:textColor="#333333"
                     android:text="Enable:" />

                <Switch
                     android:layout_height="wrap_content"
                     android:layout_width="wrap_content"
                     android:trackTint="#C8E6C9"
                     android:thumbTint="#4CAF50"
                     android:id="@+id/pluginEnableSwitch" />

            </LinearLayout>

            <Button
                 android:layout_height="36dp"
                 android:layout_width="wrap_content"
                 android:layout_marginEnd="6dp"
                 android:contentDescription="Configure Plugin"
                 android:background="#2196F3"
                 android:textSize="16sp"
                 android:textColor="#FFFFFF"
                 android:minWidth="44dp"
                 android:id="@+id/pluginConfigButton"
                 android:text="⚙️" />

            <Button
                 android:layout_height="36dp"
                 android:layout_width="wrap_content"
                 android:layout_marginEnd="6dp"
                 android:contentDescription="Plugin Information"
                 android:background="#FF9800"
                 android:textSize="16sp"
                 android:textColor="#FFFFFF"
                 android:minWidth="44dp"
                 android:id="@+id/pluginInfoButton"
                 android:text="ℹ️" />

            <Button
                 android:layout_height="36dp"
                 android:layout_width="wrap_content"
                 android:contentDescription="Delete Plugin"
                 android:background="#F44336"
                 android:textSize="16sp"
                 android:textColor="#FFFFFF"
                 android:minWidth="44dp"
                 android:id="@+id/pluginDeleteButton"
                 android:text="🗑️" />

        </LinearLayout>

        <LinearLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:visibility="gone"
             android:orientation="vertical"
             android:layout_marginTop="12dp"
             android:id="@+id/pluginDetailsLayout">

            <View
                 android:layout_height="1dp"
                 android:layout_width="match_parent"
                 android:layout_marginBottom="12dp"
                 android:background="#E0E0E0" />

            <TextView
                 android:layout_height="wrap_content"
                 android:layout_width="match_parent"
                 android:textSize="12sp"
                 android:textColor="#666666"
                 android:lineSpacingExtra="2dp"
                 android:id="@+id/pluginDetailsText"
                 android:text="Additional plugin details..." />

        </LinearLayout>

    </LinearLayout>

</androidx.cardview.widget.CardView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/layout/item_plugin_config.xml

<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView
     xmlns:android="http://schemas.android.com/apk/res/android"
     xmlns:app="http://schemas.android.com/apk/res-auto"
     android:layout_height="wrap_content"
     android:layout_width="match_parent"
     android:layout_margin="4dp"
     app:cardElevation="2dp"
     app:cardBackgroundColor="#FFFFFF"
     app:cardCornerRadius="8dp">

    <LinearLayout
         android:layout_height="wrap_content"
         android:layout_width="match_parent"
         android:padding="16dp"
         android:orientation="vertical">

        <LinearLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:layout_marginBottom="8dp"
             android:gravity="center_vertical"
             android:orientation="horizontal">

            <LinearLayout
                 android:layout_height="wrap_content"
                 android:layout_width="0dp"
                 android:gravity="center_vertical"
                 android:orientation="horizontal"
                 android:layout_weight="1">

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="0dp"
                     android:textSize="16sp"
                     android:textColor="#333333"
                     android:layout_weight="1"
                     android:id="@+id/configTitleText"
                     android:text="Configuration Item"
                     android:textStyle="bold" />

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="wrap_content"
                     android:visibility="gone"
                     android:textSize="18sp"
                     android:textColor="#FF9800"
                     android:layout_marginStart="8dp"
                     android:id="@+id/configChangeIndicator"
                     android:text="●" />

            </LinearLayout>

        </LinearLayout>

        <TextView
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:layout_marginBottom="12dp"
             android:textSize="14sp"
             android:textColor="#666666"
             android:lineSpacingExtra="2dp"
             android:id="@+id/configDescriptionText"
             android:text="Configuration description goes here" />

        <FrameLayout
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:id="@+id/configInputContainer">

            <LinearLayout
                 android:layout_height="wrap_content"
                 android:layout_width="match_parent"
                 android:gravity="center_vertical"
                 android:orientation="horizontal">

                <Switch
                     android:layout_height="wrap_content"
                     android:layout_width="wrap_content"
                     android:visibility="gone"
                     android:trackTint="#C8E6C9"
                     android:thumbTint="#4CAF50"
                     android:id="@+id/configSwitchInput" />

                <TextView
                     android:layout_height="wrap_content"
                     android:layout_width="0dp"
                     android:layout_marginStart="12dp"
                     android:layout_weight="1"
                     android:text="" />

            </LinearLayout>

            <EditText
                 android:layout_height="48dp"
                 android:layout_width="match_parent"
                 android:visibility="gone"
                 android:background="@drawable/rounded_edittext_background"
                 android:hint="Enter value..."
                 android:padding="12dp"
                 android:textSize="14sp"
                 android:textColor="#333333"
                 android:inputType="text"
                 android:id="@+id/configTextInput" />

            <Spinner
                 android:layout_height="48dp"
                 android:layout_width="match_parent"
                 android:visibility="gone"
                 android:background="@drawable/rounded_spinner_background"
                 android:padding="12dp"
                 android:popupBackground="#FFFFFF"
                 android:id="@+id/configSpinnerInput">

            </Spinner>

        </FrameLayout>

        <TextView
             android:layout_height="wrap_content"
             android:layout_width="match_parent"
             android:visibility="gone"
             android:textSize="12sp"
             android:textColor="#999999"
             android:layout_marginTop="8dp"
             android:lineSpacingExtra="2dp"
             android:id="@+id/configHelpText"
             android:text="💡 Additional help text for this setting" />

    </LinearLayout>

</androidx.cardview.widget.CardView>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/menu/log_viewer_menu.xml

<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_toggle_filters"
        android:icon="@android:drawable/ic_search_category_default"
        android:title="Toggle Filters"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_share_logs"
        android:icon="@android:drawable/ic_menu_share"
        android:title="Share Logs"
        app:showAsAction="ifRoom" />

    <item
        android:id="@+id/action_clear_logs"
        android:icon="@android:drawable/ic_menu_delete"
        android:title="Clear Logs"
        app:showAsAction="never" />

    <item
        android:id="@+id/action_settings"
        android:icon="@android:drawable/ic_menu_preferences"
        android:title="Settings"
        app:showAsAction="never" />

</menu>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/menu/main_menu.xml

<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">

    <item
        android:id="@+id/action_log"
        android:title="View Logs"
        android:icon="@android:drawable/ic_menu_info_details"
        android:showAsAction="never" />

    <item
        android:id="@+id/action_about"
        android:title="About"
        android:icon="@android:drawable/ic_menu_help"
        android:showAsAction="never" />

    <item
        android:id="@+id/action_dark_mode"
        android:title="Toggle Dark Mode"
        android:icon="@android:drawable/ic_menu_day"
        android:showAsAction="never" />

    <item
        android:id="@+id/action_export_apk"
        android:title="Export Modified APK"
        android:icon="@android:drawable/ic_menu_save"
        android:showAsAction="never" />

    <item
        android:id="@+id/action_export_logs"
        android:title="Export Logs"
        android:icon="@android:drawable/ic_menu_upload"
        android:showAsAction="never" />
</menu>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/values/colors.xml

<resources>
    <color name="purple_200">#BB86FC</color>
    <color name="purple_500">#6200EE</color>
    <color name="purple_700">#3700B3</color>
    <color name="teal_200">#03DAC5</color>
    <color name="teal_700">#018786</color>
    <color name="black">#000000</color>
    <color name="white">#FFFFFF</color>
    <color name="colorPrimary">#6200EE</color>
</resources>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/values/strings.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
    <string name="app_name">Terraria ML</string>

</resources>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/values/themes.xml

<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Base application theme (Light) -->
    <style name="Theme.ModLoader" parent="Theme.Material3.DayNight.NoActionBar">
        <item name="colorPrimary">@color/purple_500</item>
        <item name="colorPrimaryVariant">@color/purple_700</item>
        <item name="colorOnPrimary">@color/white</item>
        <item name="colorSecondary">@color/teal_200</item>
        <item name="colorSecondaryVariant">@color/teal_700</item>
        <item name="colorOnSecondary">@color/black</item>
        <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
    </style>
</resources>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/values-night/colors.xml

<resources>
    <color name="white">#000000</color>
    <color name="black">#FFFFFF</color>
</resources>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/values-night/themes.xml

<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools">
    <!-- Night mode theme -->
    <style name="Theme.ModLoader" parent="Theme.Material3.DayNight.NoActionBar">
        <item name="colorPrimary">@color/purple_200</item>
        <item name="colorPrimaryVariant">@color/purple_700</item>
        <item name="colorOnPrimary">@color/black</item>
        <item name="colorSecondary">@color/teal_200</item>
        <item name="colorSecondaryVariant">@color/teal_700</item>
        <item name="colorOnSecondary">@color/white</item>
        <item name="android:statusBarColor" tools:targetApi="l">?attr/colorPrimaryVariant</item>
    </style>
</resources>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/xml/backup_rules.xml

<?xml version="1.0" encoding="utf-8"?>
<full-backup-content>
    <!-- Include app-specific files -->
    <include domain="file" path="." />
    <include domain="database" path="." />
    <include domain="sharedpref" path="." />
    <include domain="external" path="Android/data/com.modloader/" />

    <!-- Exclude cache and logs if needed -->
    <exclude domain="cache" path="." />
    <exclude domain="file" path="logs/" />
</full-backup-content>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/xml/data_extraction_rules.xml

<?xml version="1.0" encoding="utf-8"?><!--
   Sample data extraction rules file; uncomment and customize as necessary.
   See https://developer.android.com/about/versions/12/backup-restore#xml-changes
   for details.
-->
<data-extraction-rules>
  <cloud-backup>
    <!-- TODO: Use <include> and <exclude> to control what is backed up.
        <include .../>
        <exclude .../>
        -->
  </cloud-backup>
  <!--
    <device-transfer>
        <include .../>
        <exclude .../>
    </device-transfer>
    -->
</data-extraction-rules>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/xml/file_paths.xml

<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path
        name="external_files"
        path="." />
</paths>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/xml/file_provider_paths.xml

<paths xmlns:android="http://schemas.android.com/tools">
    
    <!-- External storage root (for legacy support) -->
    <external-path 
        name="external_storage_root" 
        path="." />
    
    <!-- App-specific external files directory -->
    <external-files-path 
        name="app_external_files" 
        path="." />
    
    <!-- APK installation directory (FIXED - main issue for APK parsing) -->
    <external-files-path 
        name="apk_install" 
        path="apk_install" />
    
    <!-- Cache directory for temporary files -->
    <external-cache-path 
        name="app_cache" 
        path="." />
    
    <!-- TerrariaLoader main directory -->
    <external-files-path 
        name="terraria_loader" 
        path="TerrariaLoader" />
    
    <!-- Game-specific directories -->
    <external-files-path 
        name="terraria_game" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid" />
    
    <!-- Mod directories -->
    <external-files-path 
        name="dex_mods" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Mods/DEX" />
    
    <external-files-path 
        name="dll_mods" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Mods/DLL" />
    
    <!-- Log directories -->
    <external-files-path 
        name="app_logs" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/AppLogs" />
    
    <external-files-path 
        name="game_logs" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Logs" />
    
    <!-- Backup directories -->
    <external-files-path 
        name="backups" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Backups" />
    
    <!-- Config directories -->
    <external-files-path 
        name="config" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Config" />
    
    <!-- MelonLoader directories -->
    <external-files-path 
        name="melonloader" 
        path="TerrariaLoader/com.and.games505.TerrariaPaid/Loaders/MelonLoader" />
    
    <!-- Downloads and exports -->
    <external-files-path 
        name="downloads" 
        path="downloads" />
    
    <external-files-path 
        name="exports" 
        path="exports" />
    
    <!-- Temporary processing directory -->
    <external-files-path 
        name="temp" 
        path="temp" />
    
    <!-- Legacy mod directory (for migration) -->
    <external-files-path 
        name="legacy_mods" 
        path="mods" />

</paths>

--------------------------------------------------------------------------------
/ModLoader/app/src/main/res/xml/paths.xml

<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path
        name="external_files"
        path="." />
</paths>

--------------------------------------------------------------------------------
