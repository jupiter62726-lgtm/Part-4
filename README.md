================================================================================
ModLoader/app/src/main/res/layout/activity_setup_guide.xml

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

================================================================================
ModLoader/app/src/main/res/layout/activity_specific_selection.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#E8F5E8"
    android:fillViewport="true"
    android:padding="16dp">

    <LinearLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:gravity="center_horizontal">

        <!-- Header Section -->
        <TextView
            android:id="@+id/headerText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Choose Your Game"
            android:textSize="28sp"
            android:textStyle="bold"
            android:textColor="#2E7D32"
            android:gravity="center"
            android:layout_marginBottom="8dp"
            android:padding="16dp"
            android:background="#FFFFFF"
            android:elevation="4dp"
            android:layout_margin="8dp" />

        <TextView
            android:id="@+id/subHeaderText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Select a game to start modding with advanced tools and features"
            android:textSize="16sp"
            android:textColor="#4CAF50"
            android:gravity="center"
            android:lineSpacingExtra="4dp"
            android:layout_marginBottom="24dp"
            android:padding="12dp" />

        <!-- Available Games Section -->
        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Available for Modding"
            android:textSize="20sp"
            android:textStyle="bold"
            android:textColor="#1976D2"
            android:layout_marginBottom="16dp"
            android:gravity="center" />

        <!-- Terraria Card -->
        <androidx.cardview.widget.CardView
            android:id="@+id/terrariaCard"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="32dp"
            app:cardCornerRadius="16dp"
            app:cardElevation="8dp"
            app:cardBackgroundColor="#E8F5E8"
            android:clickable="true"
            android:focusable="true"
            android:foreground="?android:attr/selectableItemBackground">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="24dp">

                <!-- Terraria Header with Real Icon -->
                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:gravity="center_vertical"
                    android:layout_marginBottom="16dp">

                    <!-- Terraria Icon Container -->
                    <androidx.cardview.widget.CardView
                        android:layout_width="72dp"
                        android:layout_height="72dp"
                        android:layout_marginEnd="16dp"
                        app:cardCornerRadius="12dp"
                        app:cardElevation="4dp"
                        app:cardBackgroundColor="#FFFFFF">

                        <ImageView
                            android:id="@+id/terrariaIcon"
                            android:layout_width="match_parent"
                            android:layout_height="match_parent"
                            android:padding="4dp"
                            android:scaleType="centerCrop" />

                    </androidx.cardview.widget.CardView>

                    <!-- Terraria Title and Status -->
                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_height="wrap_content"
                        android:layout_weight="1"
                        android:orientation="vertical">

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Terraria"
                            android:textSize="24sp"
                            android:textStyle="bold"
                            android:textColor="#2E7D32"
                            android:layout_marginBottom="4dp" />

                        <TextView
                            android:id="@+id/terrariaStatus"
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Ready for Modding"
                            android:textSize="14sp"
                            android:textColor="#4CAF50"
                            android:textStyle="bold" />

                    </LinearLayout>

                </LinearLayout>

                <!-- Description -->
                <TextView
                    android:id="@+id/terrariaDescription"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Full-featured mod support for Terraria with both DEX/JAR and DLL mod capabilities"
                    android:textSize="16sp"
                    android:textColor="#388E3C"
                    android:layout_marginBottom="16dp"
                    android:lineSpacingExtra="2dp" />

                <!-- Features List (No Emojis) -->
                <TextView
                    android:id="@+id/terrariaFeatures"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• DEX/JAR Mods (Android Java)\n• DLL Mods (C# via MelonLoader)\n• APK Patching &amp; Installation\n• Advanced Mod Management\n• Offline Diagnostics"
                    android:textSize="14sp"
                    android:textColor="#2E7D32"
                    android:layout_marginBottom="20dp"
                    android:lineSpacingExtra="4dp" />

                <!-- Action Button -->
                <Button
                    android:id="@+id/terraria_button"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Start Modding Terraria"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:backgroundTint="#4CAF50"
                    android:textColor="#FFFFFF"
                    android:minHeight="56dp"
                    android:layout_marginTop="8dp" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Coming Soon Section -->
        <androidx.cardview.widget.CardView
            android:id="@+id/comingSoonCard"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="24dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="6dp"
            app:cardBackgroundColor="#FFF3E0"
            android:visibility="visible">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="20dp">

                <!-- Coming Soon Header -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Coming Soon"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#E65100"
                    android:layout_marginBottom="12dp" />

                <!-- Description -->
                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="We're actively working on adding support for more games. Vote for your favorite games and help us prioritize development!"
                    android:textSize="14sp"
                    android:textColor="#F57C00"
                    android:layout_marginBottom="16dp"
                    android:lineSpacingExtra="2dp" />

                <!-- Coming Soon Games Container -->
                <LinearLayout
                    android:id="@+id/comingSoonContainer"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="vertical">
                    <!-- Dynamic content will be added here -->
                </LinearLayout>

                <!-- Request Feature Button -->
                <Button
                    android:id="@+id/requestFeatureButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Request Game Support"
                    android:textSize="14sp"
                    android:backgroundTint="#FF9800"
                    android:textColor="#FFFFFF"
                    android:layout_marginTop="16dp"
                    android:minHeight="48dp" />

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Statistics Card -->
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
                    android:text="Mod Loader Statistics"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#1976D2"
                    android:layout_marginBottom="12dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="3">

                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:orientation="vertical"
                        android:gravity="center">

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="1"
                            android:textSize="20sp"
                            android:textStyle="bold"
                            android:textColor="#2196F3" />

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Supported\nGames"
                            android:textSize="12sp"
                            android:textColor="#1976D2"
                            android:gravity="center" />

                    </LinearLayout>

                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:orientation="vertical"
                        android:gravity="center">

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="2"
                            android:textSize="20sp"
                            android:textStyle="bold"
                            android:textColor="#2196F3" />

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Mod\nFormats"
                            android:textSize="12sp"
                            android:textColor="#1976D2"
                            android:gravity="center" />

                    </LinearLayout>

                    <LinearLayout
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:orientation="vertical"
                        android:gravity="center">

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="∞"
                            android:textSize="20sp"
                            android:textStyle="bold"
                            android:textColor="#2196F3" />

                        <TextView
                            android:layout_width="wrap_content"
                            android:layout_height="wrap_content"
                            android:text="Mod\nPossibilities"
                            android:textSize="12sp"
                            android:textColor="#1976D2"
                            android:gravity="center" />

                    </LinearLayout>

                </LinearLayout>

            </LinearLayout>

        </androidx.cardview.widget.CardView>

        <!-- Navigation and Info Section -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:gravity="center">

            <!-- Back Button -->
            <Button
                android:id="@+id/backToMainButton"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="← Back to Main Menu"
                android:textSize="16sp"
                android:backgroundTint="@android:color/transparent"
                android:textColor="#666666"
                android:layout_marginBottom="16dp"
                android:minHeight="48dp" />

            <!-- Info Text -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="More games and features are added regularly. Check for updates!"
                android:textSize="12sp"
                android:textColor="#888888"
                android:gravity="center"
                android:background="#F0F0F0"
                android:padding="12dp"
                android:layout_marginBottom="16dp" />

            <!-- Version Info -->
            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="ModLoader v1.0 - Enhanced Game Selection"
                android:textSize="11sp"
                android:textColor="#999999"
                android:gravity="center"
                android:layout_marginBottom="16dp" />

        </LinearLayout>

    </LinearLayout>

</ScrollView>

================================================================================
ModLoader/app/src/main/res/layout/activity_terraria_specific_updated.xml

<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:fillViewport="true"
    android:scrollbars="vertical">

    <LinearLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp"
        android:background="#2E2E2E">

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
                android:background="#3A3A3A"
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
                    android:backgroundTint="#4CAF50"
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
                        android:backgroundTint="#81C784"
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
                        android:backgroundTint="#A5D6A7"
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
                        android:backgroundTint="#2196F3"
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
                        android:backgroundTint="#FF9800"
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

        <!-- Plugin Management Section (NEW) -->
        <androidx.cardview.widget.CardView
            android:id="@+id/pluginCard"
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
                    android:text="🧩 Plugin System"
                    android:textSize="20sp"
                    android:textStyle="bold"
                    android:textColor="#7B1FA2"
                    android:layout_marginBottom="16dp" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal"
                    android:weightSum="2">

                    <Button
                        android:id="@+id/pluginManagerButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="🔧 Manage Plugins"
                        android:textSize="14sp"
                        android:backgroundTint="#9C27B0"
                        android:textColor="@android:color/white"
                        android:layout_marginEnd="6dp"
                        android:minHeight="48dp" />

                    <Button
                        android:id="@+id/pluginInstallButton"
                        android:layout_width="0dp"
                        android:layout_weight="1"
                        android:layout_height="wrap_content"
                        android:text="📥 Install Plugin"
                        android:textSize="14sp"
                        android:backgroundTint="#BA68C8"
                        android:textColor="@android:color/white"
                        android:layout_marginStart="6dp"
                        android:minHeight="48dp" />
                </LinearLayout>

                <Button
                    android:id="@+id/pluginConfigButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="⚙️ Plugin Configuration"
                    android:textSize="14sp"
                    android:backgroundTint="#CE93D8"
                    android:textColor="#4A148C"
                    android:layout_marginTop="12dp"
                    android:minHeight="48dp" />

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="• Extend app functionality with custom plugins\n• Supports JAR/DEX plugin files"
                    android:textSize="12sp"
                    android:textColor="#8E24AA"
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
            app:cardBackgroundColor="#FFF8E1">

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
                    android:textColor="#F57C00"
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
                        android:backgroundTint="#FF9800"
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
                        android:backgroundTint="#FFA726"
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
                        android:backgroundTint="#FFB74D"
                        android:textColor="#E65100"
                        android:layout_marginStart="4dp"
                        android:minHeight="44dp" />
                </LinearLayout>

                <Button
                    android:id="@+id/diagnosticButton"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🧪 Offline Diagnostic &amp; Repair"
                    android:textSize="14sp"
                    android:textStyle="bold"
                    android:backgroundTint="#FF8F00"
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
            android:background="#3A3A3A"
            android:padding="16dp"
            android:layout_marginTop="16dp"
            android:layout_marginBottom="16dp" />

    </LinearLayout>
</ScrollView>

================================================================================
ModLoader/app/src/main/res/layout/activity_unified_loader.xml

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

================================================================================
ModLoader/app/src/main/res/layout/activity_universal.xml

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

================================================================================
ModLoader/app/src/main/res/layout/addon_config_dialog.xml

<!-- File: addon_config_dialog.xml -->
<!-- Path: app/src/main/res/layout/addon_config_dialog.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="16dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="⚙️ Addon Configuration"
        android:textSize="20sp"
        android:textStyle="bold"
        android:textColor="#333333"
        android:layout_marginBottom="16dp" />

    <ScrollView
        android:layout_width="match_parent"
        android:layout_height="300dp"
        android:background="#F5F5F5"
        android:padding="8dp">

        <LinearLayout
            android:id="@+id/configOptionsContainer"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <!-- Configuration options will be added dynamically here -->

        </LinearLayout>

    </ScrollView>

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:layout_marginTop="16dp"
        android:gravity="end">

        <Button
            android:id="@+id/resetConfigButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="🔄 Reset"
            android:background="#FF9800"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/saveConfigButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="💾 Save"
            android:background="#4CAF50"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/cancelConfigButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="❌ Cancel"
            android:background="#F44336"
            android:textColor="#FFFFFF" />

    </LinearLayout>

</LinearLayout>


================================================================================
ModLoader/app/src/main/res/layout/addon_creation_dialog.xml

<!-- File: addon_creation_dialog.xml -->
<!-- Path: app/src/main/res/layout/addon_creation_dialog.xml -->
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:orientation="vertical"
    android:padding="20dp">

    <TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="✨ Create New Addon"
        android:textSize="24sp"
        android:textStyle="bold"
        android:textColor="#2E7D32"
        android:gravity="center"
        android:layout_marginBottom="20dp" />

    <!-- Addon Basic Info -->
    <androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:elevation="4dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="📋 Basic Information"
                android:textSize="16sp"
                android:textStyle="bold"
                android:layout_marginBottom="12dp" />

            <com.google.android.material.textfield.TextInputLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="8dp">

                <EditText
                    android:id="@+id/addonNameInput"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="Addon Name"
                    android:inputType="text" />

            </com.google.android.material.textfield.TextInputLayout>

            <com.google.android.material.textfield.TextInputLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="8dp">

                <EditText
                    android:id="@+id/addonIdInput"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="Addon ID (lowercase, underscores only)"
                    android:inputType="text" />

            </com.google.android.material.textfield.TextInputLayout>

            <com.google.android.material.textfield.TextInputLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginBottom="8dp">

                <EditText
                    android:id="@+id/addonDescriptionInput"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:hint="Description"
                    android:inputType="textMultiLine"
                    android:lines="3" />

            </com.google.android.material.textfield.TextInputLayout>

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal">

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:layout_marginEnd="8dp">

                    <EditText
                        android:id="@+id/addonVersionInput"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:hint="Version (e.g., 1.0.0)"
                        android:inputType="text"
                        android:text="1.0.0" />

                </com.google.android.material.textfield.TextInputLayout>

                <com.google.android.material.textfield.TextInputLayout
                    android:layout_width="0dp"
                    android:layout_height="wrap_content"
                    android:layout_weight="1"
                    android:layout_marginStart="8dp">

                    <EditText
                        android:id="@+id/addonAuthorInput"
                        android:layout_width="match_parent"
                        android:layout_height="wrap_content"
                        android:hint="Author"
                        android:inputType="text" />

                </com.google.android.material.textfield.TextInputLayout>

            </LinearLayout>

        </LinearLayout>
    </androidx.cardview.widget.CardView>

    <!-- Category Selection -->
    <androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="16dp"
        android:elevation="4dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="🏷️ Category"
                android:textSize="16sp"
                android:textStyle="bold"
                android:layout_marginBottom="12dp" />

            <Spinner
                android:id="@+id/addonCategorySpinner"
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:background="#F5F5F5" />

        </LinearLayout>
    </androidx.cardview.widget.CardView>

    <!-- Template Selection -->
    <androidx.cardview.widget.CardView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginBottom="20dp"
        android:elevation="4dp">

        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical"
            android:padding="16dp">

            <TextView
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:text="📄 Template"
                android:textSize="16sp"
                android:textStyle="bold"
                android:layout_marginBottom="12dp" />

            <RadioGroup
                android:id="@+id/templateRadioGroup"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical">

                <RadioButton
                    android:id="@+id/basicTemplateRadio"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="📝 Basic Template (empty configuration)"
                    android:checked="true" />

                <RadioButton
                    android:id="@+id/themeTemplateRadio"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🎨 Theme Template (colors and styles)" />

                <RadioButton
                    android:id="@+id/utilityTemplateRadio"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🛠️ Utility Template (file operations)" />

                <RadioButton
                    android:id="@+id/customTemplateRadio"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="⚙️ Custom Template (advanced configuration)" />

            </RadioGroup>

        </LinearLayout>
    </androidx.cardview.widget.CardView>

    <!-- Action Buttons -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="end">

        <Button
            android:id="@+id/previewAddonButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="👁️ Preview"
            android:background="#2196F3"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/createAddonButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="✨ Create"
            android:background="#4CAF50"
            android:textColor="#FFFFFF"
            android:layout_marginEnd="8dp" />

        <Button
            android:id="@+id/cancelCreateButton"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="❌ Cancel"
            android:background="#F44336"
            android:textColor="#FFFFFF" />

    </LinearLayout>

</LinearLayout>

================================================================================
ModLoader/app/src/main/res/layout/dialog_log_settings.xml

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

================================================================================
ModLoader/app/src/main/res/layout/item_addon.xml

<!-- File: item_addon.xml -->
<!-- Path: app/src/main/res/layout/item_addon.xml -->
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/addonCardView"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="8dp"
    app:cardElevation="4dp"
    app:cardBackgroundColor="#FFFFFF"
    android:clickable="true"
    android:focusable="true"
    android:foreground="?android:attr/selectableItemBackground">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- Header Row -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="center_vertical"
            android:layout_marginBottom="8dp">

            <LinearLayout
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:orientation="vertical">

                <TextView
                    android:id="@+id/addonNameText"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Addon Name"
                    android:textSize="18sp"
                    android:textStyle="bold"
                    android:textColor="#333333"
                    android:layout_marginBottom="4dp" />

                <!-- ADD: Missing addon_name for legacy compatibility -->
                <TextView
                    android:id="@+id/addon_name"
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Legacy Name Field"
                    android:textSize="14sp"
                    android:textColor="#666666"
                    android:visibility="gone" />

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:orientation="horizontal">

                    <TextView
                        android:id="@+id/addonVersionText"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="v1.0.0"
                        android:textSize="12sp"
                        android:textColor="#666666"
                        android:background="#E0E0E0"
                        android:padding="4dp"
                        android:layout_marginEnd="8dp" />

                    <TextView
                        android:id="@+id/addonCategoryText"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="Category"
                        android:textSize="12sp"
                        android:textColor="#FFFFFF"
                        android:background="#2196F3"
                        android:padding="4dp" />

                </LinearLayout>

            </LinearLayout>

            <Switch
                android:id="@+id/addonEnabledSwitch"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginStart="16dp"
                android:thumbTint="#4CAF50"
                android:trackTint="#E0E0E0" />

            <!-- ADD: Missing addon_toggle for legacy compatibility -->
            <Switch
                android:id="@+id/addon_toggle"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:visibility="gone" />

        </LinearLayout>

        <!-- Description -->
        <TextView
            android:id="@+id/addonDescriptionText"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="Addon description goes here..."
            android:textSize="14sp"
            android:textColor="#666666"
            android:layout_marginBottom="12dp"
            android:maxLines="3"
            android:ellipsize="end" />

        <!-- Action Buttons -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:gravity="end">

            <Button
                android:id="@+id/configureButton"
                android:layout_width="wrap_content"
                android:layout_height="32dp"
                android:text="⚙️ Config"
                android:textSize="12sp"
                android:background="#2196F3"
                android:textColor="#FFFFFF"
                android:layout_marginEnd="4dp"
                android:minWidth="70dp" />

            <Button
                android:id="@+id/exportButton"
                android:layout_width="wrap_content"
                android:layout_height="32dp"
                android:text="📤 Export"
                android:textSize="12sp"
                android:background="#FF9800"
                android:textColor="#FFFFFF"
                android:layout_marginEnd="4dp"
                android:minWidth="70dp" />

            <Button
                android:id="@+id/deleteButton"
                android:layout_width="wrap_content"
                android:layout_height="32dp"
                android:text="🗑️ Delete"
                android:textSize="12sp"
                android:background="#F44336"
                android:textColor="#FFFFFF"
                android:minWidth="70dp" />

        </LinearLayout>

    </LinearLayout>

</androidx.cardview.widget.CardView>


================================================================================
ModLoader/app/src/main/res/layout/item_log_entry.xml

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

================================================================================
ModLoader/app/src/main/res/layout/item_mod.xml

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


================================================================================
ModLoader/app/src/main/res/menu/addon_management_menu.xml

<!-- File: addon_management_menu.xml -->
<!-- Path: app/src/main/res/menu/addon_management_menu.xml -->
<menu xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <item
        android:id="@+id/action_statistics"
        android:title="📊 Statistics"
        android:icon="@android:drawable/ic_menu_info_details"
        app:showAsAction="never" />

    <item
        android:id="@+id/action_cleanup"
        android:title="🧹 Cleanup"
        android:icon="@android:drawable/ic_menu_delete"
        app:showAsAction="never" />

    <item
        android:id="@+id/action_help"
        android:title="❓ Help"
        android:icon="@android:drawable/ic_menu_help"
        app:showAsAction="never" />

</menu>

================================================================================
ModLoader/app/src/main/res/menu/log_viewer_menu.xml

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

================================================================================
ModLoader/app/src/main/res/menu/main_menu.xml

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

================================================================================
ModLoader/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

================================================================================
ModLoader/app/src/main/res/mipmap-anydpi-v26/ic_launcher_round.xml

<?xml version="1.0" encoding="utf-8"?>
<adaptive-icon xmlns:android="http://schemas.android.com/apk/res/android">
    <background android:drawable="@drawable/ic_launcher_background" />
    <foreground android:drawable="@drawable/ic_launcher_foreground" />
</adaptive-icon>

================================================================================
ModLoader/app/src/main/res/values/colors.xml

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


================================================================================
ModLoader/app/src/main/res/values/strings.xml

<?xml version="1.0" encoding="utf-8"?>
<resources>
   <string name="app_name">Terraria ML</string>
   <string name="plugin_service_description">Background service for addon operations</string>
   <string name="permission_plugin_api_label">Plugin API Access</string>
   <string name="permission_plugin_api_desc">Allows addon to access plugin APIs</string>
   <string name="permission_plugin_install_label">Plugin Installation</string>
   <string name="permission_plugin_install_desc">Allows installation of new plugins</string>
   <string name="permission_plugin_manage_label">Plugin Management</string>
   <string name="permission_plugin_manage_desc">Allows management of existing plugins</string>
   <string name="addon_management_title">Addon Management</string>
   <string name="addon_statistics">Statistics</string>
   <string name="addon_cleanup">Cleanup</string>
   <string name="addon_help">Help</string>
</resources>


================================================================================
ModLoader/app/src/main/res/values/themes.xml

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

================================================================================
ModLoader/app/src/main/res/values-night/colors.xml

<resources>
    <color name="white">#000000</color>
    <color name="black">#FFFFFF</color>
</resources>

================================================================================
ModLoader/app/src/main/res/values-night/themes.xml

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

================================================================================
ModLoader/app/src/main/res/xml/backup_rules.xml

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

================================================================================
ModLoader/app/src/main/res/xml/data_extraction_rules.xml

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

================================================================================
ModLoader/app/src/main/res/xml/file_paths.xml

<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path
        name="external_files"
        path="." />
</paths>

================================================================================
ModLoader/app/src/main/res/xml/file_provider_paths.xml

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

================================================================================
ModLoader/app/src/main/res/xml/paths.xml

<?xml version="1.0" encoding="utf-8"?>
<paths xmlns:android="http://schemas.android.com/apk/res/android">
    <external-files-path
        name="external_files"
        path="." />
</paths>
