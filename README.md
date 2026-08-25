# Draw

A drawing and colouring canvas for kids. One HTML file wrapped in a Capacitor
Android shell.

## Layout

    www/index.html   the entire app - UI, canvas, flood fill, photo background
    android/         generated native project (committed, safe to edit)
    .gitlab-ci.yml   builds a debug APK on every push

## Getting a test APK

Push the repo. Both CI configs are included - use whichever host you are on,
and delete the other.

**GitHub** (`.github/workflows/android.yml`)

    Actions tab > click the newest run > scroll to the bottom >
    Artifacts > click "draw-debug-..."

**GitLab** (`.gitlab-ci.yml`)

    CI/CD > Pipelines > click the job > Download artifacts (right panel)

Either way you get a zip. Unzip it for `app-debug.apk`. On the phone, allow
installs from unknown sources, then open the file.

GitHub runs take about 4-5 minutes. GitLab takes 6-8 on the first run because
it downloads the Android SDK, then 2-3 once the cache is warm.

## Changing the app

Edit `www/index.html`, commit, push. Nothing else needs touching - the CI
runs `npx cap sync android` before every build.

## Running it locally

    npm install
    npx cap sync android
    npx cap open android        # needs Android Studio

Or just open `www/index.html` in a browser. Everything works there except
Save, which falls back to a normal file download instead of the share sheet.

## Before the first Play Store upload

1. **Change the application ID.** It is `com.drawcanvas.app` right now and it
   is permanent once published. It appears in `capacitor.config.json`,
   `android/app/build.gradle` (`namespace` and `applicationId`), and the
   package folder under `android/app/src/main/java/`.
2. **Replace the launcher icons** in `android/app/src/main/res/mipmap-*/`.
3. **Set the app name** in `android/app/src/main/res/values/strings.xml`.
4. **Build a signed release**, not this debug APK. Play needs an `.aab`:
   `./gradlew bundleRelease` with a keystore configured. Never commit the
   keystore - put it in GitLab CI/CD variables as a base64 file variable.
5. **Read the Families policy.** Apps aimed at children have extra rules on
   ads, data collection, and privacy policies, even fully offline ones.

## Optional: bundle the font

The UI asks for Fredoka and falls back to the system font when it is absent.
To ship it, download the woff2 files from Google Fonts and drop them in
`www/fonts/` as `Fredoka-Regular.woff2`, `Fredoka-Medium.woff2`, and
`Fredoka-SemiBold.woff2`. No code change needed.

## Tuning

Constants at the top of the script in `www/index.html`:

    FILL_TOLERANCE     how far a colour may drift and still be filled.
                       Raise it if fills leave a halo around lines, lower it
                       if colour leaks through thin outlines.
    MAX_CANVAS_EDGE    caps the canvas pixel buffer.
    MAX_UNDO           number of undo snapshots kept. Each one is a full-size
                       bitmap, so this times the canvas size is the memory
                       ceiling. Lower both if you see crashes on cheap devices.
