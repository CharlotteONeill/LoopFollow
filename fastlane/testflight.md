# Using Github Actions + FastLane to deploy to TestFlight: the "Browser Build" method

These instructions allow you to build this Open-Source app without having access to a Mac.

* You can install the app on phones using TestFlight that are not connected to your computer
* You can send builds and updates to those you care for
* You can install the app on your phone using only the TestFlight app if a phone was lost or the app is accidentally deleted
* You do not need to worry about specific Xcode/Mac versions for a given iOS

## **Automatic Builds**
>
> The browser build defaults to automatically updating and building a new version of the app according to this schedule:
>
> * automatically checks for updates weekly on Wednesdays and if updates are found, it will build a new version of the app
> * automatically builds once a month regardless of whether there are updates on the first of the month
> * with each scheduled run (weekly or monthly), a successful Build log appears - if the time is very short, it did not need to build - only the long actions (>5 minutes) built a new app
>
> It also creates an alive branch, if you don't already have one. See [Why do I have an alive branch?](#why-do-i-have-an-alive-branch).
>
> The [**Optional**](#optional) section provides instructions to modify the default behavior if desired.

## Introduction

The setup steps are somewhat involved, but nearly all are one time steps. Subsequent builds are trivial. Your app must be updated once every 90 days, but it's a simple click to make a new build and can be done from anywhere.

Note that installing with TestFlight requires the Apple ID account holder for the phone be 13 years or older (age varies with country). This can be circumvented by logging into Media & Purchase on the child's phone with an adult's account. More details on this can be found in [LoopDocs](https://loopkit.github.io/loopdocs/gh-actions/gh-deploy/#install-testflight-loop-for-child).

This method for building without a Mac was ported from Loop. If you have used this method for Loop or one of the other DIY apps (LoopCaregiver, Trio, xDrip4iOS), some of the steps can be re-used and the full set of instructions does not need to be repeated. This will be mentioned in relevant sections below.

There are more detailed instructions in LoopDocs for doing Browser Builds of Loop and other apps, including troubleshooting and build errors. Please refer to [LoopDocs](https://loopkit.github.io/loopdocs/gh-actions/gh-other-apps/) for more details.

If you build multiple apps, it is strongly recommended that you configure a free *GitHub* organization and do all your building in the organization. This means you enter items one time for the organization (6 SECRETS required to build and 1 VARIABLE required to automatically update your certificates annually). Otherwise, those 6 SECRETS must be entered for every repository. Please refer to [LoopDocs: Use a *GitHub* Organization Account](https://loopkit.github.io/loopdocs/gh-actions/gh-other-apps/#use-a-github-organization-account).

## Prerequisites

* A [github account](https://github.com/signup). The free level comes with plenty of storage and free compute time to build several apps, multiple times a day, if you wanted to.
* A paid [Apple Developer account](https://developer.apple.com).
* Some time. Set aside a couple of hours to perform the setup.
* Use the same GitHub account for all "Browser Builds" of the various DIY apps.

## Save 6 Secrets

You require 6 Secrets (alphanumeric items) to use the GitHub build method and if you use the GitHub method to build more than one app, you will use the same 6 Secrets for each app. (Examples are Loop, LoopCaregiver, LoopFollow, Trio and xDrip4iOS).

Each secret is indentified below by `ALL_CAPITAL_LETTER_NAMES`.

* Four Secrets are from your Apple Account
* Two Secrets are from your GitHub account
* Be sure to save the 6 Secrets in a text file using a text editor
  * Do **NOT** use a smart editor, which might auto-correct and change case, because these Secrets are case sensitive

Refer to [LoopDocs: Make a Secrets Reference File](https://loopkit.github.io/loopdocs/gh-actions/gh-first-time/#make-a-secrets-reference-file) for a handy template to use when saving your Secrets.

## Generate App Store Connect API Key

This step is common for all GitHub Browser Builds; do this step only once.

> If you already built an app using this method, reuse the 4 Secrets. If this is your first app, you will be saving 4 Secrets from your Apple Account in this step.

1. Sign in to the [Apple developer portal page](https://developer.apple.com/account/resources/certificates/list).
1. Copy the Team ID from the upper right of the screen. Record this as your `TEAMID`.
1. Go to the [App Store Connect](https://appstoreconnect.apple.com/access/integrations/api) interface, click the "Integrations" tab, and create a new key with "Admin" access. Give it the name: "FastLane API Key".
1. Record the issuer id; this will be used for `FASTLANE_ISSUER_ID`.
1. Record the key id; this will be used for `FASTLANE_KEY_ID`.
1. Download the API key itself, and open it in a text editor. The contents of this file will be used for `FASTLANE_KEY`. Copy the full text, including the "-----BEGIN PRIVATE KEY-----" and "-----END PRIVATE KEY-----" lines.

## Create GitHub Personal Access Token

If you have previously built another app using the "browser build" method, you use the same personal access token (`GH_PAT`), so skip this step. If you use a free GitHub organization to build, you still use the same personal access token. This is created using your personal GitHub username.

Log into your GitHub account to create a personal access token; this is one of two GitHub secrets needed for your build.

1. Create a [new personal access token](https://github.com/settings/tokens/new):
    * Enter a name for your token, use "FastLane Access Token".
    * Change the Expiration selection to `No expiration`.
    * Select the `workflow` permission scope \* this also selects `repo` scope.
    * Click "Generate token".
    * Copy the token and record it. It will be used below as `GH_PAT`.

## Make up a Password

This is the second one of two GitHub secrets needed for your build.

The first time you build with the GitHub Browser Build method for any DIY app, you will make up a password and record it as `MATCH_PASSWORD`. You use the same password for all DIY apps. Note, if you later lose `MATCH_PASSWORD`, you will need to delete your Match-Secrets repository (automatically created), and go through the GitHub actions again.

## GitHub Match-Secrets Repository

> A private Match-Secrets repository is automatically created under your GitHub username the first time you run a GitHub Action. Because it is a private repository - only you can see it. You will not take any direct actions with this repository; it needs to be there for GitHub to use as you progress through the steps.

## Setup Github LoopFollow repository

1. Fork https://github.com/loopandlearn/LoopFollow into your GitHub username (using your organization if you have one). If you already have a fork of LoopFollow in GitHub, you should not make another one. Do not rename the repository. You can continue to work with your existing fork, or delete that from GitHub and then fork again.
1. If you are using an organization, do this step at the organization level, e.g., username-org. If you are not using an organization, do this step at the repository level, e.g., username/LoopFollow:
    * Go to Settings -> Secrets and variables -> Actions and make sure the Secrets tab is open
1. For each of the following secrets, tap on "New organization secret" or "New repository secret", then add the name of the secret, along with the value you recorded for it:
    * `TEAMID`
    * `FASTLANE_ISSUER_ID`
    * `FASTLANE_KEY_ID`
    * `FASTLANE_KEY`
    * `GH_PAT`
    * `MATCH_PASSWORD`
1. If you are using an organization, do this step at the organization level, e.g., username-org. If you are not using an organization, do this step at the repository level, e.g., username/LoopFollow:
    * Go to Settings -> Secrets and variables -> Actions and make sure the Variables tab is open
1. Tap on "Create new organization variable" or "Create new repository variable", then add the name below and enter the value true. Unlike secrets these variables are visible and can be edited.
    * `ENABLE_NUKE_CERTS`

## Validate Repository Secrets

This step validates most of your six Secrets and provides error messages if it detects an issue with one or more. In addition, if you do not have a private Match-Secrets repository it creates one for you.

1. Click on the "Actions" tab of your LoopFollow repository.
1. Select "1. Validate Secrets".
1. Click "Run Workflow", and tap the green button.
1. Wait, and within a minute or two you should see a green checkmark indicating the workflow succeeded.
1. The workflow will check if the required secrets are added and that they are correctly formatted. If errors are detected, please check the run log for details.

## Add Identifiers for LoopFollow App

1. Click on the "Actions" tab of your LoopFollow repository.
1. Select "2. Add Identifiers".
1. Click "Run Workflow", and tap the green button.
1. Wait, and within a minute or two you should see a green checkmark indicating the workflow succeeded.

## Create LoopFollow App in App Store Connect

If you have created a LoopFollow app in App Store Connect before, skip ahead to [Create Building Certificates](#create-building-certificates).

1. Go to the [apps list](https://appstoreconnect.apple.com/apps) on App Store Connect and click the blue "plus" icon to create a New App.
    * Select "iOS".
    * Select a name: this will have to be unique, so you may have to try a few different names here, but it will not be the name you see on your phone, so it's not that important.
    * Select your primary language.
    * Choose the bundle ID that matches `com.TEAMID.LoopFollow`, with TEAMID matching your team id.
    * SKU can be anything; e.g. "123".
    * Select "Full Access".
1. Click Create

You do not need to fill out the next form. That is for submitting to the app store.

## Create Building Certificates

This step is no longer required. The Build LoopFollow function now takes care of this for you. It does not hurt to run it but is not needed.

Once a year, you will get an email from Apple indicating your certificate will expire in 30 days. You can ignore that email. When it does expire, the next time an automatic or manual build happens, the expired certificate information will be removed (nuked) from your Match-Secrets repository and a new one created. This should happen without you needing to take any action.

## Build LoopFollow

1. Click on the "Actions" tab of your LoopFollow repository.
1. Select "4. Build LoopFollow".
1. Click "Run Workflow", select your branch, and tap the green button.
1. You have some time now. Go enjoy a coffee. The build should take about 15 minutes.
1. Your app should eventually appear on [App Store Connect](https://appstoreconnect.apple.com/apps).
1. For each phone/person you would like to support LoopFollow on:
    * Add them in [Users and Access](https://appstoreconnect.apple.com/access/users) on App Store Connect.
    * Add them to your TestFlight Internal Testing group.

## TestFlight and Deployment Details

For more details, please refer to [LoopDocs: Set Up Users](https://loopkit.github.io/loopdocs/gh-actions/gh-first-time/#set-up-users-and-access-testflight) and [LoopDocs: Deploy](https://loopkit.github.io/loopdocs/gh-actions/gh-deploy/)

## Automatic Build FAQs

### Why do I have an `alive` branch?

If a GitHub repository has no activity (no commits are made) in 60 days, then GitHub disables the ability to use automated actions for that repository. We need to take action more frequently than that or the automated build process won't work.

The `build_LoopFollow.yml` file uses a special branch called `alive` and adds a dummy commit to the `alive` branch at regular intervals. This "trick" keeps the Actions enabled so the automated build works.

The branch `alive` is created automatically for you. Do not delete or rename it! Do not modify `alive` yourself; it is not used for building the app. (There are actually 2 alive branches: `alive-main` and `alive-dev`; they both serve a similar purpose and should be left alone.)

## OPTIONAL

What if you don't want to allow automated updates of the repository or automatic builds?

You can affect the default behavior:

1. [`GH_PAT` `workflow` permission](#gh_pat-workflow-permission)
1. [Modify scheduled building and synchronization](#modify-scheduled-building-and-synchronization)

### `GH_PAT` `workflow` permission

To enable the scheduled build and sync, the `GH_PAT` must hold the `workflow` permission scopes. This permission serves as the enabler for automatic and scheduled builds with browser build. To verify your token holds this permission, follow these steps.

1. Go to your [FastLane Access Token](https://github.com/settings/tokens)
2. It should say `repo`, `workflow` next to the `FastLane Access Token` link
3. If it does not, click on the link to open the token detail view
4. Click to check the `workflow` box. You will see that the checked boxes for the `repo` scope become disabled (change color to dark gray and are not clickable)
5. Scroll all the way down to and click the green `Update token` button
6. Your token now holds both required permissions

If you choose not to have automatic building enabled, be sure the `GH_PAT` has `repo` scope or you won't be able to manually build.

### Modify scheduled building and synchronization

You can modify the automation by creating and using some variables.

To configure the automated build more granularly involves creating up to two environment variables: `SCHEDULED_BUILD` and/or `SCHEDULED_SYNC`. See [How to configure a variable](#how-to-configure-a-variable).

Note that the weekly and monthly Build Trio actions will continue, but the actions are modified if one or more of these variables is set to false. **A successful Action Log will still appear, even if no automatic activity happens**.

* If you want to manually decide when to update your repository to the latest commit, but you want the monthly builds and keep-alive to continue: set `SCHEDULED_SYNC` to false and either do not create `SCHEDULED_BUILD` or set it to true
* If you want to only build when an update has been found: set `SCHEDULED_BUILD` to false and either do not create `SCHEDULED_SYNC` or set it to true
    * **Warning**: if no updates to your default branch are detected within 90 days, your previous TestFlight build may expire requiring a manual build

|`SCHEDULED_SYNC`|`SCHEDULED_BUILD`|Automatic Actions|
|---|---|---|
| `true` (or NA) | `true` (or NA) | keep-alive, weekly update check (auto update/build), monthly build with auto update |
| `true` (or NA) | `false` | keep-alive, weekly update check with auto update, only builds if update detected |
| `false` | `true` (or NA) | keep-alive, monthly build, no auto update |
| `false` | `false` | no automatic activity, no keep-alive |

### How to configure a variable

1. Go to the "Settings" tab of your Trio repository.
2. Click on `Secrets and Variables`.
3. Click on `Actions`
4. You will now see a page titled *Actions secrets and variables*. Click on the `Variables` tab
5. To disable ONLY scheduled building, do the following:
    * Click on the green `New repository variable` button (upper right)
    * Type `SCHEDULED_BUILD` in the "Name" field
    * Type `false` in the "Value" field
    * Click the green `Add variable` button to save.
6. To disable scheduled syncing, add a variable:
    * Click on the green `New repository variable` button (upper right)
    * Type `SCHEDULED_SYNC` in the "Name" field
    * Type `false` in the "Value" field
    * Click the green `Add variable` button to save

Your build will run on the following conditions:

* Default behaviour:
  * Run weekly, every Wednesday to check for changes; if there are changes, it will update your repository and build
  * Run monthly, every first of the month, if there are changes, it will update your repository; regardless of changes, it will build
  * Each time the action runs, it makes a keep-alive commit to the `alive` branch if necessary
* If you disable any automation (both variables set to `false`), no updates, keep-alive or building happens when Build Trio runs
* If you disabled just scheduled synchronization (`SCHEDULED_SYNC` set to`false`), it will only run once a month, on the first of the month, no update will happen; keep-alive will run
* If you disabled just scheduled build (`SCHEDULED_BUILD` set to `false`), it will run once weekly, every Wednesday, to check for changes; if there are changes, it will update and build; keep-alive will run

## What if I build using more than one GitHub username

This is not typical. But if you do use more than one GitHub username, follow these steps at the time of the annual certificate renewal.

1. After the certificates were removed (nuked) from username1 Match-Secrets storage, you need to switch to username2
1. Add the variable FORCE_NUKE_CERTS=true to the username2/Trio repository
1. Run the action Create Certificate (or Build, but Create is faster)
1. Immediately set FORCE_NUKE_CERTS=false or delete the variable

Now certificates for username2 have been cleared out of Match-Secrets storage for username2. Building can proceed as usual for both username1 and username2.
