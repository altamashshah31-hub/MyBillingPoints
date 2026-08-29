name: Build My Billing Points APK

on:
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout
        uses: actions/checkout@v4

      - name: Set up JDK 17
        uses: actions/setup-java@v4
        with:
          distribution: 'temurin'
          java-version: '17'

      - name: Set up Gradle
        uses: gradle/actions/setup-gradle@v4
        with:
          gradle-version: '8.7'

      - name: Extract Android project
        run: |
          mkdir -p build-project
          unzip -qo MyBillingPoints_APK_BuildReady.zip -d build-project
          # Fix KSP version
          find build-project -type f \( -name "build.gradle.kts" -o -name "build.gradle" \) -exec sed -i 's/2.0.21-1.0.28/2.0.21-1.0.27/g' {} +
          # Fix JVM target compatibility
          find build-project -type f \( -name "build.gradle.kts" -o -name "build.gradle" \) -exec sed -i 's/JavaVersion.VERSION_1_8/JavaVersion.VERSION_17/g' {} +
          find build-project -type f \( -name "build.gradle.kts" -o -name "build.gradle" \) -exec sed -i 's/jvmTarget = "1.8"/jvmTarget = "17"/g' {} +

      - name: Build APK
        working-directory: build-project
        run: gradle assembleDebug --no-daemon -Dkotlin.jvm.target.validation.mode=warning

      - name: Upload APK
        uses: actions/upload-artifact@v4
        with:
          name: MyBillingPoints-APK
          path: build-project/**/build/outputs/apk/debug/*.apk
          
