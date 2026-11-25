# release-lint demo

A blank `create-expo-app` (Expo SDK 58, React Native 0.88) with six **deliberate** release mistakes, to try
[release-lint](https://github.com/harpreetkbadhan/release-lint) in a few minutes. Every value in it is fake.

| Where | Mistake | Caught by |
|---|---|---|
| `src/planted-mistakes.ts` | A Stripe live secret key in the JS bundle | `secret/stripe-secret-key` |
| `src/planted-mistakes.ts` | A staging API URL | `host/non-production` |
| `src/planted-mistakes.ts` | A dev server on the local network | `host/private-network` |
| `app.json` → `extra.apiUrl` | `http://localhost:3000` in the Expo config | `host/localhost` |
| `app.json` → `android.intentFilters` | An App Link to `staging.example.com` | `deeplink/non-production-host` |
| `app.json` → `ios.associatedDomains` | `applinks:staging.example.com` | `deeplink/non-production-host` |

None of these show up in a code review of a single file, and the last three only exist in the final app after Expo
generates the native projects. That is the point of scanning the built file.

## Try it in 30 seconds

The release build is attached to the [v1.0.0 release](https://github.com/harpreetkbadhan/release-lint-demo/releases/tag/v1.0.0):

```sh
curl -LO https://github.com/harpreetkbadhan/release-lint-demo/releases/download/v1.0.0/release-lint-demo.aab
npx app-release-lint release-lint-demo.aab
```

## Try it (Android)

Needs Node 20.19+, JDK 17+ and the Android SDK.

```sh
npm ci
npx expo prebuild --platform android
(cd android && ./gradlew bundleRelease)
npx app-release-lint android/app/build/outputs/bundle/release/app-release.aab
```

## Try it (iOS)

Needs your own Apple signing:

```sh
npx eas-cli build --local --platform ios --profile production --output app.ipa
npx app-release-lint app.ipa
```

## In CI

[`.github/workflows/release-lint.yml`](.github/workflows/release-lint.yml) builds the `.aab` on GitHub Actions and runs
the [release-lint Action](https://github.com/harpreetkbadhan/release-lint#in-ci) on it. It only runs when started by hand
(Actions → release-lint → Run workflow), and it **fails on purpose**: the findings appear as annotations on the run and
as a table in the job summary. That red run is what a release mistake looks like in your pipeline.

Only scan apps you own or have permission to audit.

## License

MIT
