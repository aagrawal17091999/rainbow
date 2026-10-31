# Install audit: aagrawal17091999/rainbow

Run on 2026-10-31 against `develop` at `d07ed58`. No analytics tool is set in `.datalyze/config.yaml`, so this audit used the code alone. Every finding cites the code (file:line) or a query in the snapshot. Each one was checked by an independent verifier (claude-opus-5-5) that tried to disprove it: confirmed findings are below, undecided ones are open questions, disproved ones are listed at the end. Verifier: 110 confirmed, 6 undecided, 4 disproved.

## Summary

The code sends **227 named events** and **37 user properties** (227 client), through `analytics.track` from `@/analytics`, `trackSwapEvent` from `@/__swaps__/utils/trackSwapEvent`, `analytics.track`, `analytics.track`, `analytics.screen`, `analytics.track`, `analytics.screen` from `@/analytics`, `analytics.track`, `analytics.screen` from `@/analytics`, `analytics.screen`. All of them are now in the registry, drafted from the code and completed by reading it.

| | Count |
|---|---|
| Events in the registry | 227 |
| User properties | 37 |
| Code-level problems (identity, SDK calls, unreadable names) | 1116 |
| Legacy findings accepted in the baseline | 2054 |

**Health: not measured.** This audit ran without analytics data, so nothing here says what arrives; the code-level checks still ran.

**Fix first:**
1. **Raw wallet addresses and ENS names sent as event properties, bypassing the wallet-address hash** (F1)
2. **Raw token-search text is sent as an event property** (F2)
3. **Free-text and user-entered values are sent as event properties** (F3)

**Not checked:** 1 part of this audit couldn't be finished; the report lists them under "What wasn't checked". Running the first audit again retries them.

## How this product tracks today

- **Stack:** typescript (expo), kotlin (android) in `android`, swift (ios) in `ios`; SDKs: `posthog-react-native` (client). All of it is in `_tracking_profile.yaml`, which the PR check reads to write new calls in this style.
- **SDKs called directly:** none outside the wrappers.
- **Wrappers and other tracking functions:** `analytics.track` (`@/analytics`), adds `walletAddressHash`, `walletType`: src/analytics/index.ts: Analytics.track wraps PostHog capture(event, {...defaultMetadata, ...walletContext, ...params}). Event names are analytics.event.* / event.* constants from src/analytics/event.ts. Used from ~150 files.; `analytics.identify` (`@/analytics`), identify: src/analytics/index.ts: Analytics.identify calls PostHog identify(deviceId, {walletAddressHash, walletType, ...userProperties}); the id is the device id, the argument is traits only.; `trackSwapEvent` (`@/__swaps__/utils/trackSwapEvent`), adds `degenMode`, `errorMessage`, `inputAssetAddress`, `inputAssetAmount`, `inputAssetChainId`, `inputAssetName`, `inputAssetSymbol`, `inputAssetType`, `isBridge`, `isHardwareWallet`, `isSwappingToPopularAsset`, `mainnetAddress`, `outputAssetAddress`, `outputAssetAmount`, `outputAssetChainId`, `outputAssetName`, `outputAssetSymbol`, `outputAssetType`, `tradeAmountUSD`, `type`: src/__swaps__/utils/trackSwapEvent.ts: second-level wrapper. The caller passes the event constant (swapsSubmitted or swapsFailed) as arg 0. The wrapper builds every property itself from the swaps store and the call's parameters, then calls analytics.track(event, params).; `analytics.track`, adds `walletAddressHash`, `walletType`: Every file imports the instance as a named export (`import { analytics } from '@/analytics'`, e.g. Chart.tsx:14). The earlier entry with `from` didn't match that import shape, so this one matches `analytics.track` by its callee alone. src/analytics/index.ts:102 is `track(event, params, walletContext)`, which sends the PostHog `capture`.; `analytics.identify`, identify: Same named-instance import as above. src/analytics/index.ts:77 `identify(userProperties)` calls `client.identify(deviceId, traits)`. Chart.tsx:271 and :282 pass traits such as `{ chartType }`.; `analytics.track`, adds `walletAddressHash`, `walletType`: src/analytics/index.ts exports `analytics = new Analytics()`, a named export holding an instance, not an SDK default or namespace import. This entry has no `from`, so it matches `analytics.track(...)` by name in case the `from: "@/analytics"` entry doesn't match the named-export instance. That entry matched none of the calls above, which are flagged as vendor SDK calls outside a wrapper.; `analytics.screen`: src/analytics/index.ts:91 `screen(route, params)` sends a PostHog screen event named by a Routes value. Callers pass the route constant, so the scanner may read it from Routes.; `analytics.identify` (`@/analytics`), user_set: src/analytics/index.ts identify(userProperties) sends them as PostHog person traits via client.identify(deviceId, {...metadata, ...userProperties}); callers like src/features/positions/stores/positionsStore.ts:167 and src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:41 use it only to set profile properties (positionsAmount, stakedRnbw, rnbwMembershipTier); `analytics.track`, adds `walletAddressHash`, `walletType`: src/analytics/index.ts exports `analytics = new Analytics()` (a named-import class instance, wrapping PostHog.capture). The `from: @/analytics` entry did not match member calls on that named import, so every analytics.track(...) was still flagged as a vendor SDK call. This entry matches the call by its text.; `analytics.identify`, identify: src/analytics/index.ts Analytics.identify wraps PostHog.identify(deviceId, traits). src/hooks/useIdentifyAppIcon.ts:10 calls analytics.identify({ appIcon }) and was flagged as a vendor SDK call for the same reason as analytics.track.; `analytics.screen` (`@/analytics`), adds `walletAddressHash`, `walletType`: src/analytics/index.ts defines screen(route, params) which calls PostHog screen; src/navigation/onNavigationStateChange.js:130 calls it with the route name; `analytics.track`, adds `walletAddressHash`, `walletType`: src/analytics/index.ts exports `analytics` as a named export (a Analytics class instance), so the scanner isn't matching the named import against `from`. This entry matches `analytics.track` as written in code, and the callers' own flagged lines (wallet.ts, settings.ts, notifications/analytics.ts) read it. track() wraps posthog capture and always adds walletAddressHash and walletType.; `analytics.identify`, identify: src/analytics/index.ts Analytics.identify(userProperties) calls posthog identify(deviceId, traits). Same named-export reason as analytics.track, and it covers notifications/analytics.ts:24,118 and redux/settings.ts:115.; `analytics.screen` (`@/analytics`), adds `walletAddressHash`, `walletType`: src/analytics/index.ts:91-95 screen(route, params) spreads getDefaultMetadata() (walletAddressHash, walletType) into every screen event; it adds nothing else on iOS. Only caller is src/navigation/onNavigationStateChange.js:130.; `analytics.screen`, adds `walletAddressHash`, `walletType`: Same wrapper method as above, for call sites the scanner can't tie to the import..
- **Wrappers** (allowed to call the SDK, not scanned): `src/analytics/index.ts`, `src/__swaps__/utils/trackSwapEvent.ts`.
- **Client vs. server:** 227 client, 0 server.
- **Identity:** `identify` (On every app launch right after the device id is read or created (src/App.tsx:173), and again, with no traits, each time a wallet is initialized or switched, to attach walletAddressHash and walletType (src/state/wallets/initializeWallet.ts:135). Traits are then sent from many places as state changes.), with Device id from getOrCreateDeviceId(), kept in the keychain. It is not a user id, email or wallet address. Wallet identity goes in as a hashed address trait (walletAddressHash), not as the id.; no reset (none in the code).
- **Naming:** 222 of 227 events don't match the naming pattern (`^[a-z][a-z0-9]*(_[a-z0-9]+)+$`). They keep their names in the registry, because that's what the data holds; renames are in the migration plan.
- **Coverage:** code indexed 315 lines that import an analytics SDK or look like tracking, in 9 pattern groups across 2705 source files: 295 read by the scanner, 1 SDK imports, 4 inside the wrappers, 2 calls to another analytics tool, 8 explained, 5 not tracking. Every one is accounted for.
- **Other analytics tools in the code** (not audited; the audit covers the tool in .datalyze/config.yaml): SentrySDK.capture (2 lines, e.g. `ios/AppDelegate.swift:102`).
- **Looked like tracking, isn't:** `reads.track` (createSelectorReadTracker is a state-store selector read tracker used in tests, not analytics.); `chartReadTracker.track` (The chart data store tracks which chart ids are being read for subscriptions (createSelectorReadTracker). It sends no analytics.); `pnlShareImageRef.current.capture` (A view-shot screenshot capture for sharing a PnL image, not tracking.).
- **What the scanner can't see:** 
  - `src/__swaps__/screens/Swap/components/FlipButton.tsx:97` (dynamic): Calls the wrapper as analytics.track(analytics.event.swapsFlippedAssets, ...). The name is a member of the event map in src/analytics/event.ts, which maps swapsFlippedAssets to 'swaps.flipped_assets'. It is one fixed name, not built from data.
  - `src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:75` (dynamic): analytics.track(analytics.event.swapsChangedChainId, ...). The map in src/analytics/event.ts:189 gives 'swaps.changed_chain_id'. Only the 'type' property varies (input or output).
  - `src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:173` (dynamic): analytics.track(analytics.event.swapsSearchedForToken, ...) with type 'output'. The map in src/analytics/event.ts:188 gives 'swaps.searched_for_token'.
  - `src/__swaps__/screens/Swap/components/TokenList/TokenToSellList.tsx:88` (dynamic): analytics.track(analytics.event.swapsSearchedForToken, ...) with type 'input'. It is the same event as TokenToBuyList.tsx:173.
  - `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:8` (dynamic): analyticsTrack is a local alias of analytics.track so it can be passed to runOnJS. Its only caller, line 47 of the same file, passes analytics.event.swapsQuoteFailed, which is 'swaps.quote_failed' (src/analytics/event.ts:196). The one name is credited here.
  - `src/__swaps__/screens/Swap/hooks/useSearchCurrencyLists.ts:161` (dynamic): analytics.track(analytics.event.tokenList, params). The map in src/analytics/event.ts:225 gives 'token_list'. The effect is throttled and only runs for queries longer than 2 characters.
  - `src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438` (dynamic): analytics.track(analytics.event.swapsReceivedQuote, ...). The map in src/analytics/event.ts:192 gives 'swaps.received_quote'.
  - `src/__swaps__/screens/Swap/hooks/useSwapSettings.ts:14` (dynamic): analytics.track(analytics.event.swapsToggledDegenMode, { enabled }). The map in src/analytics/event.ts:191 gives 'swaps.toggled_degen_mode'.
  - `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023` (dynamic): analytics.track(analytics.event.swapsSelectedAsset, ...). The map in src/analytics/event.ts:187 gives 'swaps.selected_asset'.
  - `src/__swaps__/screens/Swap/providers/SyncSwapStateAndSharedValues.tsx:111` (dynamic): analytics.track(analytics.event.insufficientNativeAssetForAction, { type: Routes.SWAP, nativeAssetSymbol }). The map in src/analytics/event.ts:198 gives 'insufficient_native_asset_for_action'.
  - `src/App.tsx:173` (wrapper): analytics.identify({ installSource }) is the wrapper's own identify imported from '@/analytics' (src/analytics/index.ts:77), which calls PostHog identify with the device id. It is not a direct SDK call.
  - `src/App.tsx:177` (dynamic): analytics.track(analytics.event.debugIdentifyProbe, ...) passes a key of the `event` map in src/analytics/event.ts, and the wrapper sends the map's value. The name is a fixed string, not data.
  - `src/App.tsx:197` (wrapper): analytics.identify({ screenHeight, screenWidth, screenScale }) goes through the '@/analytics' wrapper (src/analytics/index.ts:77). It sets user traits on the first app open.
  - `src/App.tsx:198` (dynamic): analytics.track(analytics.event.firstAppOpen) passes a key of the `event` map in src/analytics/event.ts:79.
  - `src/app/error-boundary/ErrorBoundary.tsx:14` (dynamic): analytics.track(analytics.event.errorBoundaryReset, ...) passes a key of the `event` map (src/analytics/event.ts:218).
  - `src/app/error-boundary/ErrorBoundary.tsx:21` (dynamic): analytics.track(analytics.event.errorBoundary, ...) passes a key of the `event` map (src/analytics/event.ts:217).
  - `src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88` (dynamic): analytics.track(analytics.event.claimablePanelOpened, ...) passes a key of the `event` map (src/analytics/event.ts:214).
  - `src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23` (dynamic): analytics.track(analytics.event.pressedButton, { buttonName, action }) passes a key of the `event` map (src/analytics/event.ts:81).
  - `src/components/asset-list/RecyclerAssetList2/index.tsx:102` (dynamic): analytics.track(analytics.event.navigationMyQrCode, ...) passes a key of the `event` map (src/analytics/event.ts:289).
  - `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140` (dynamic): analytics.track(analytics.event.navigationAddCash, ...) passes a key of the `event` map (src/analytics/event.ts:286).
  - `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140` (dynamic): analytics.track(analytics.event.navigationAddCash, ...) goes through the wrapper (src/analytics/index.ts, Analytics.track, which calls PostHog capture). The name is a key of the `event` object in src/analytics/event.ts (navigationAddCash: 'Tapped "Add Cash"'). The scanner can't resolve the member access, but the value is a literal.
  - `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157` (dynamic): analytics.event.navigationSwap resolves to the literal in src/analytics/event.ts:287.
  - `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174` (dynamic): analytics.event.navigationSend resolves to the literal in src/analytics/event.ts:288.
  - `src/components/cards/EthCard.tsx:74` (dynamic): analytics.event.buyButtonPressed resolves to the literal in src/analytics/event.ts:91.
  - `src/components/cards/EthCard.tsx:90` (dynamic): analytics.event.cardPressed resolves to the literal in src/analytics/event.ts:87.
  - `src/components/cards/LearnCard.tsx:31` (dynamic): analytics.event.cardPressed resolves to the literal in src/analytics/event.ts:87.
  - `src/components/cards/ReceiveAssetsCard.tsx:47` (dynamic): analytics.event.qrCodeViewed resolves to the literal in src/analytics/event.ts:90.
  - `src/components/Discover/DiscoverScreenContext.tsx:64` (dynamic): analytics.event.discoverTapSearch resolves to the literal in src/analytics/event.ts:300.
  - `src/components/Discover/DiscoverSearch.tsx:117` (dynamic): analytics.event.tokenList resolves to the literal in src/analytics/event.ts:225. The properties come from a local `params` object (screen, no_icon, no_price, total_tokens, query).
  - `src/components/Discover/DiscoverSearchInput.tsx:134` (dynamic): analytics.event.searchQuery resolves to the literal in src/analytics/event.ts:42.
  - `src/components/Discover/useTrackDiscoverScreenTime.ts:22` (dynamic): This file imports `event` from '@/analytics/event' directly and passes event.timeSpentOnDiscoverScreen, which is the literal at src/analytics/event.ts:250.
  - `src/components/expanded-state/NewWalletGroupState.tsx:20` (dynamic): analytics.event.addNewWalletGroupName resolves to the literal in src/analytics/event.ts:184.
  - `src/components/expanded-state/UniqueTokenExpandedState.tsx:389` (dynamic): analytics.event.tokenDetailsNFT resolves to the literal in src/analytics/event.ts:222.
  - `src/components/expanded-state/WalletProfileState.js:54` (dynamic): The name is `analytics.event.walletProfileCancelled`, a key of the constant `event` map in src/analytics/event.ts:292. It is a fixed string.
  - `src/components/expanded-state/WalletProfileState.js:61` (dynamic): The name is `analytics.event.walletProfileSubmitted`, which is a fixed string in src/analytics/event.ts:293.
  - `src/components/sheet/sheet-action-buttons/BuyActionButton.tsx:32` (dynamic): The name is `analytics.event.buyButtonPressed` (src/analytics/event.ts:91). It is sent after navigating to the add-cash route. The same name is also sent from src/components/cards/EthCard.tsx:74.
  - `src/components/value-chart/Chart.tsx:272` (dynamic): The name is `event.chartTypeChanged` (src/analytics/event.ts:314). It is sent from a `useListen` on `chartType` in ChartsTelemetry.
  - `src/components/value-chart/Chart.tsx:283` (dynamic): The name is `event.candleResolutionChanged` (src/analytics/event.ts:315). It is sent from a `useListen` on `candleResolution`.
  - `src/features/app-icon/components/AppIconSettingsSection.tsx:32` (dynamic): The name is `analytics.event.setAppIcon` (src/analytics/event.ts:60). It is sent when a person picks an icon.
  - `src/features/app-icon/screens/AppIconUnlockSheet.tsx:39` (dynamic): The name is `analytics.event.appIconUnlockSheetCTAPressed` (src/analytics/event.ts:179).
  - `src/features/app-icon/screens/AppIconUnlockSheet.tsx:43` (dynamic): The name is `analytics.event.appIconUnlockSheetViewed` (src/analytics/event.ts:178). It is sent from a mount effect.
  - `src/features/backup/backup.ts:187` (dynamic): The name is `analytics.event.backupError` (src/analytics/event.ts:271). It is sent from the catch block of the cloud backup.
  - `src/features/backup/backup.ts:532` (dynamic): The name is `analytics.event.backupSavedPassword` (src/analytics/event.ts:272). It is sent after the iCloud shared-credentials save succeeds.
  - `src/features/backup/backup.ts:535` (dynamic): The name is `analytics.event.backupSkippedPassword` (src/analytics/event.ts:273). It is sent from the catch block when the iCloud save fails.
  - `src/features/backup/components/BackupCloudStep.tsx:58` (dynamic): The name is `analytics.event.backupChoosePassword` (src/analytics/event.ts:277). It is sent from a mount effect.
  - `src/features/backup/components/BackupCloudStep.tsx:58` (dynamic): analytics.track(analytics.event.backupChoosePassword): the name is a key of the `event` map in src/analytics/event.ts:277, not a literal at the call; the scanner can't read analytics.event.<key>.
  - `src/features/backup/components/BackupSheetSection.tsx:56` (dynamic): Name is analytics.event.backupSheetShown, a key of the event map (src/analytics/event.ts:276).
  - `src/features/backup/hooks/useCreateBackup.ts:65` (dynamic): Name is analytics.event.backupComplete (src/analytics/event.ts:274).
  - `src/features/backup/hooks/useCreateBackup.ts:92` (dynamic): Name is analytics.event.backupConfirmed (src/analytics/event.ts:275).
  - `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477` (dynamic): Name is analytics.event.cashAmountEntered (src/analytics/event.ts:128).
  - `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:495` (dynamic): Name is analytics.event.cashWalletCheckFailed (src/analytics/event.ts:132).
  - `src/features/cash/screens/add-cash-sheet/ReauthenticateContent.tsx:51` (dynamic): Name is analytics.event.cashLostPasskeyRecoverySelected (src/analytics/event.ts:138).
  - `src/features/cash/screens/add-wallet-sheet/AddWalletSheet.tsx:87` (dynamic): Name is analytics.event.cashWalletLinkPrompted (src/analytics/event.ts:150).
  - `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36` (dynamic): Name is analytics.event.cashWalletLinked (src/analytics/event.ts:151).
  - `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:47` (dynamic): Name is analytics.event.cashWalletLinkFailed (src/analytics/event.ts:152).
  - `src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40` (dynamic): A ternary on the remote flag cash_signup_enabled picks between two keys of the event map (src/analytics/event.ts:126-127).
  - `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47` (dynamic): Name is analytics.event.cashPasskeySubmitted (src/analytics/event.ts:145).
  - `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55` (dynamic): Name is analytics.event.cashPasskeyAdded (src/analytics/event.ts:146).
  - `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73` (dynamic): Name is analytics.event.cashPasskeyFailed (src/analytics/event.ts:147). Not in the list yet, but the same file and pattern.
  - `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73` (dynamic): analytics.track(analytics.event.cashPasskeyFailed, ...) passes the name through the `event` map in src/analytics/event.ts, a fixed key-to-string map (line 147). It is not built from data. The call goes through the wrapper in src/analytics/index.ts, which calls posthog capture itself.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68` (dynamic): The name is analytics.event.cashPhoneSubmitted, a constant in the event map in src/analytics/event.ts:133. It is sent from advanceToChallenge after the challenge starts, with the property `mode`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78` (dynamic): The name is analytics.event.cashPhoneSubmitFailed, a constant in src/analytics/event.ts:134. It is sent in the catch of advanceToChallenge, with the property `reason`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126` (dynamic): The name is analytics.event.cashPhoneAlreadyRegistered, a constant in src/analytics/event.ts:136. It is sent with `outcome: 'alreadyRegistered'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:134` (dynamic): The name is analytics.event.cashPhoneAlreadyRegistered, the same constant as line 126 (src/analytics/event.ts:136). It is sent with `outcome: 'registeredWithPasskey'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:154` (dynamic): The name is analytics.event.cashPhoneSubmitFailed, a constant in src/analytics/event.ts:134. It is sent in the catch around createUserWithPhone, with the property `reason`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:190` (dynamic): The name is analytics.event.cashExistingAccountRecoverySelected, a constant in src/analytics/event.ts:137. It is sent with no properties, in chooseRecovery.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91` (dynamic): The name is analytics.event.cashPhoneVerified, a constant in src/analytics/event.ts:139. It is sent with `mode: 'recovery'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101` (dynamic): The name is analytics.event.cashPhoneVerifyFailed, a constant in src/analytics/event.ts:140. It is sent with `mode: 'recovery'` and `reason: 'client_error'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` (dynamic): The name is analytics.event.cashPhoneSubmitted, a constant in src/analytics/event.ts:133. It is sent with `mode: 'resume'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:155` (dynamic): The name is analytics.event.cashKycAwaitingDecision, a constant in src/analytics/event.ts:143. It is sent once per run, with `source: 'submit'`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:176` (dynamic): The name is analytics.event.cashKycSubmitted, a constant in src/analytics/event.ts:141. It is sent with no properties, before submitOnboarding.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:191` (dynamic): The name is analytics.event.cashKycFailed, a constant in src/analytics/event.ts:144. It is sent in the submitOnboarding catch, with the property `reason`.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:191` (dynamic): analytics.track(analytics.event.cashKycFailed, ...) goes through the app's wrapper (src/analytics/index.ts, which calls PostHog capture). The name is a key of the `event` map in src/analytics/event.ts:144, not a runtime value: cashKycFailed = 'cash.kyc_failed'. The line is flagged as a vendor call only because `analytics.event.*` is not a literal.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:227` (dynamic): analytics.event.cashKycApproved resolves to the constant in src/analytics/event.ts:142. It is sent through the wrapper when the KYC outcome is 'approved'.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:231` (dynamic): analytics.event.cashKycFailed (src/analytics/event.ts:144), sent with reason 'state_not_supported' when the outcome is unsupportedState.
  - `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:235` (dynamic): analytics.event.cashKycFailed (src/analytics/event.ts:144), sent with reason 'rejected' in the default branch of the switch.
  - `src/features/cash/services/cashSignInService.ts:53` (dynamic): analytics.event.cashSignInSubmitted is the constant at src/analytics/event.ts:153, sent with { trigger } when the login ceremony starts.
  - `src/features/cash/services/cashSignInService.ts:68` (dynamic): analytics.event.cashSignInSucceeded is the constant at src/analytics/event.ts:154, sent with { trigger } after the token is stored.
  - `src/features/cash/services/cashSignInService.ts:76` (dynamic): analytics.event.cashSignInCancelled is the constant at src/analytics/event.ts:156, sent with { trigger } when the passkey prompt is cancelled.
  - `src/features/cash/services/cashSignInService.ts:78` (dynamic): analytics.event.cashSignInFailed is the constant at src/analytics/event.ts:155, sent with { trigger, reason } for failures that are not cancellations or access refusals.
  - `src/features/cash/services/kycStatusService.ts:22` (dynamic): analytics.event.cashKycApproved (src/analytics/event.ts:142), sent when the outcome is 'approved'.
  - `src/features/cash/services/kycStatusService.ts:23` (dynamic): analytics.event.cashKycAwaitingDecision (src/analytics/event.ts:143), sent with { source } when the outcome is 'reviewing'.
  - `src/features/cash/services/kycStatusService.ts:24` (dynamic): analytics.event.cashKycFailed (src/analytics/event.ts:144), sent with reason 'rejected'.
  - `src/features/cash/services/kycStatusService.ts:25` (dynamic): analytics.event.cashKycFailed (src/analytics/event.ts:144), sent with reason 'state_not_supported' in the final else.
  - `src/features/cash/stores/cardLinkFlowStore.ts:47` (dynamic): analytics.event.cashCardLinked is the constant at src/analytics/event.ts:148, sent with { brand } once the card is linked.
  - `src/features/cash/stores/cardLinkFlowStore.ts:58` (dynamic): analytics.track(analytics.event.cashCardLinkFailed, ...): the name is a member of the `event` constant map in src/analytics/event.ts:149, a fixed string. The call goes through the analytics wrapper (src/analytics/index.ts), which wraps PostHog.
  - `src/features/cash/stores/cashBuyOrderStore.ts:123` (dynamic): analytics.track(analytics.event.cashBuyOrderCompleted, ...): fixed name from the event map (src/analytics/event.ts:130), sent through the wrapper.
  - `src/features/cash/stores/cashBuyOrderStore.ts:146` (dynamic): analytics.track(analytics.event.cashBuyOrderFailed, ...): fixed name from the event map (src/analytics/event.ts:131).
  - `src/features/cash/stores/cashBuyOrderStore.ts:233` (dynamic): analytics.track(analytics.event.cashBuyOrderFailed, ...) on a definitive createBuyOrder rejection: same fixed name (src/analytics/event.ts:131).
  - `src/features/cash/stores/cashBuyOrderStore.ts:263` (dynamic): analytics.track(analytics.event.cashBuyOrderSubmitted, ...): fixed name from the event map (src/analytics/event.ts:129).
  - `src/features/cash/stores/verifyPhoneFlowStore.ts:106` (dynamic): analytics.track(analytics.event.cashPhoneSubmitted, { mode: 'recovery' }): fixed name from the event map (src/analytics/event.ts:133).
  - `src/features/cash/stores/verifyPhoneFlowStore.ts:127` (dynamic): analytics.track(analytics.event.cashPhoneVerified, { mode }): fixed name from the event map (src/analytics/event.ts:139).
  - `src/features/cash/stores/verifyPhoneFlowStore.ts:155` (dynamic): analytics.track(analytics.event.cashPhoneVerifyFailed, { reason, mode }): fixed name from the event map (src/analytics/event.ts:140).
  - `src/features/cash/stores/verifyPhoneFlowStore.ts:194` (dynamic): analytics.track(analytics.event.cashPhoneResendFailed, { reason, mode }): fixed name from the event map (src/analytics/event.ts:135).
  - `src/features/dapp-browser/components/Homepage.tsx:226` (dynamic): analytics.track(analytics.event.browserTapFavorite, dapp): fixed name from the event map (src/analytics/event.ts:203). The properties are the FavoritedSite object, typed at event.ts:795.
  - `src/features/dapp-browser/components/Homepage.tsx:334` (dynamic): analytics.track(analytics.event.browserTrendingDappClicked, { name, url, hasClickedBefore, index }): fixed name from the event map (src/analytics/event.ts:201).
  - `src/features/dapp-browser/stores/favoriteDappsStore.ts:50` (dynamic): analytics.track(analytics.event.browserAddFavorite, { url, image, name }): fixed name from the event map (src/analytics/event.ts:202).
  - `src/features/dapp-request/hooks/useTransactionSetup.ts:55` (dynamic): analytics.track(analytics.event.txRequestShownSheet, { source }): fixed name from the event map (src/analytics/event.ts:181).
  - `src/features/dapp-request/hooks/useTransactionSetup.ts:55` (dynamic): analytics.track(analytics.event.txRequestShownSheet, ...). The name is a key of the `event` const object in src/analytics/event.ts:181, not a string literal. The value is 'request.sheet.show'.
  - `src/features/dapp-request/screens/SignTransactionSheet.tsx:295` (dynamic): analytics.event.txRequestReject, which is 'request.rejected' (src/analytics/event.ts:182).
  - `src/features/dapp-request/screens/SignTransactionSheet.tsx:406` (dynamic): analytics.event.txRequestApprove, which is 'request.approved' (src/analytics/event.ts:183). This is the transaction-approval path.
  - `src/features/dapp-request/screens/SignTransactionSheet.tsx:511` (dynamic): analytics.event.txRequestApprove, which is 'request.approved' (src/analytics/event.ts:183). This is the signature-approval path.
  - `src/features/discover/components/DiscoverHeader.tsx:134` (dynamic): analyticsEvent.discoverTabPressed, where analyticsEvent is `event` imported from '@/analytics/event'. The value is 'discover.tab_pressed' (event.ts:253).
  - `src/features/discover/components/markets/layouts/MarketCarousel.tsx:87` (dynamic): event.discoverPredictionOrderPressed, which is 'discover.prediction_order_pressed' (event.ts:252).
  - `src/features/discover/components/markets/layouts/MarketGrid.tsx:83` (dynamic): event.discoverPredictionOrderPressed, which is 'discover.prediction_order_pressed' (event.ts:252).
  - `src/features/discover/components/markets/layouts/MarketList.tsx:82` (dynamic): event.discoverPredictionOrderPressed, which is 'discover.prediction_order_pressed' (event.ts:252).
  - `src/features/discover/components/PredictionEventsSection.tsx:87` (dynamic): analyticsEvent.discoverSectionPressed, which is 'discover.section_pressed' (event.ts:254).
  - `src/features/discover/components/SectionLayout.tsx:41` (dynamic): event.discoverSectionPressed, which is 'discover.section_pressed' (event.ts:254).
  - `src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17` (dynamic): event.discoverCarouselScrolled, which is 'discover.carousel_scrolled' (event.ts:255). The call is debounced by 500 ms.
  - `src/features/discover/utils/trackDiscoverCardPress.ts:28` (dynamic): event.discoverCardPressed, which is 'discover.card_pressed' (event.ts:251). This helper has no name argument. It always sends this one event, and its callers (MarketCarousel, MarketGrid, MarketList) pass the properties.
  - `src/features/ens/components/ENSCreateProfileCard.tsx:51` (dynamic): analytics.event.cardPressed, which is 'card.pressed' (event.ts:87).
  - `src/features/ens/components/ENSSearchCard.tsx:54` (dynamic): analytics.track(analytics.event.cardPressed, ...) goes through the wrapper in src/analytics/index.ts. The name is a key of the event map in src/analytics/event.ts:87, which the scanner doesn't follow through the analytics.event member. It is not a direct SDK call.
  - `src/features/perps/screens/ClosePositionBottomSheet.tsx:150` (dynamic): Name is analytics.event.perpsClosedPosition (src/analytics/event.ts:321), one fixed string.
  - `src/features/perps/screens/ClosePositionBottomSheet.tsx:150` (dynamic): analytics.track(analytics.event.perpsClosedPosition, ...) is a call through the wrapper. The name is a key of the `event` object in src/analytics/event.ts:321, which the scanner can't follow through the class field `event = event` on Analytics. The value is a fixed string.
  - `src/features/perps/screens/ClosePositionBottomSheet.tsx:179` (dynamic): Sent from the catch block when closePosition throws. The name is the constant event.perpsClosePositionFailed (event.ts:322).
  - `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268` (dynamic): Sent after createTriggerOrder succeeds. The name is the constant event.perpsTriggerOrderCreated (event.ts:323).
  - `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281` (dynamic): Sent from the catch block of createTriggerOrder. The name is the constant event.perpsTriggerOrderFailed (event.ts:324).
  - `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60` (dynamic): Sent after cancelOrder succeeds. The name is the constant event.perpsTriggerOrderCanceled (event.ts:325).
  - `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71` (dynamic): Sent from the catch block of cancelOrder. The name is the constant event.perpsTriggerOrderCancelFailed (event.ts:326).
  - `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214` (dynamic): Sent when createIsolatedMarginPosition returns a falsy result. The name is the constant event.perpsOpenPositionCanceled (event.ts:319). The registry will need to cover it as a name sent from this sheet.
  - `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228` (dynamic): Sent after the margin position is created. The name is the constant event.perpsAddedToPosition (event.ts:329).
  - `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242` (dynamic): Sent from the catch block. The name is the constant event.perpsAddedToPositionFailed (event.ts:330).
  - `src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73` (dynamic): Sent after the share sheet closes. The name is the constant event.perpsTradeDetailsSharePressed (event.ts:331).
  - `src/features/perps/withdrawalConfig.ts:48` (dynamic): Sent after the Hyperliquid withdraw call succeeds. The name is the constant event.perpsWithdrew (event.ts:327).
  - `src/features/perps/withdrawalConfig.ts:57` (dynamic): Sent from the catch block of the withdrawal. The name is the constant event.perpsWithdrawFailed (event.ts:328).
  - `src/features/placements/engagement/trackInteraction.ts:20` (wrapper): trackPlacementInteraction always sends event.placementInteraction ('placement.interaction', event.ts:256). It is now a tracker of its own, so callers are read with their properties. trackSurfaceInteraction in the same file (line 43) sends 'surface.interaction' (event.ts:257) and is also a tracker.
  - `src/features/placements/engagement/trackInteraction.ts:43` (dynamic): `event.surfaceInteraction` is a key in the `event` const map in src/analytics/event.ts:257. The map is `as const`, so the name is always the literal 'surface.interaction'.
  - `src/features/polymarket/depositConfig.ts:59` (dynamic): `analytics.event.predictionsDepositFailed` is read through the `event` instance field on Analytics (src/analytics/index.ts:31, `event = event`). It maps to src/analytics/event.ts:346. The name is fixed. `metadata` is DepositFailureMetadata from '@/systems/funding/types', passed straight through.
  - `src/features/polymarket/depositConfig.ts:60` (dynamic): `analytics.event.predictionsDeposit` maps to src/analytics/event.ts:345. The name is fixed. `metadata` is DepositSuccessMetadata, passed straight through.
  - `src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:209` (dynamic): `analytics.event.predictionsPlaceOrderFailed` maps to src/analytics/event.ts:342. It fires in the catch block of the buy handler and sends side: 'buy'.
  - `src/features/polymarket/screens/polymarket-sell-position-sheet/PolymarketSellPositionSheet.tsx:127` (dynamic): `analytics.event.predictionsPlaceOrderFailed` maps to src/analytics/event.ts:342. It fires in the catch block of the sell handler and sends side: 'sell'. The same event is also sent from the buy sheet.
  - `src/features/polymarket/utils/executePolymarketOrder.ts:324` (dynamic): `analytics.event.predictionsPlaceOrder` maps to src/analytics/event.ts:341. The name is fixed. It is sent from trackMatchedOrderAnalytics.
  - `src/features/polymarket/utils/executePolymarketOrder.ts:353` (dynamic): `analytics.event.predictionsOrderMatchFailed` maps to src/analytics/event.ts:349. The name is fixed. It is sent from trackOrderMatchFailedAnalytics.
  - `src/features/polymarket/utils/redeemPosition.ts:59` (dynamic): `analytics.event.predictionsRedeemPosition` maps to src/analytics/event.ts:343. The name is fixed. It is sent from trackRedeemPosition after the relayer transaction succeeds.
  - `src/features/polymarket/utils/redeemPosition.ts:82` (dynamic): `analytics.event.predictionsRedeemPositionFailed` maps to src/analytics/event.ts:344. The name is fixed. It is sent from trackRedeemPositionFailed in the catch block.
  - `src/features/polymarket/withdrawalConfig.ts:62` (dynamic): `analytics.event.predictionsWithdrawalFailed` maps to src/analytics/event.ts:348. The name is fixed. `metadata` is WithdrawalFailureMetadata, passed through.
  - `src/features/polymarket/withdrawalConfig.ts:63` (dynamic): `analytics.event.predictionsWithdrawal` maps to src/analytics/event.ts:347. The name is fixed. `metadata` is WithdrawalSuccessMetadata, passed through.
  - `src/features/positions/components/PositionCard.tsx:78` (dynamic): `analytics.event.positionsOpenedSheet` maps to src/analytics/event.ts:173. The name is fixed. It fires on press of a position card, before navigating to the position sheet.
  - `src/features/positions/screens/PositionSheet.tsx:68` (dynamic): `analytics.event.positionsOpenedExternalDapp` maps to src/analytics/event.ts:174. The name is fixed. It fires when the person opens the dapp URL in the browser.
  - `src/features/positions/screens/PositionSheet.tsx:68` (dynamic): The name is analytics.event.positionsOpenedExternalDapp, a key of the event map in src/analytics/event.ts:174. It is a fixed string, 'Viewed external dapp', and the call goes through the wrapper analytics.track, not a vendor SDK.
  - `src/features/positions/screens/PositionSheet.tsx:81` (dynamic): The name is analytics.event.positionsOpenedAsset, which is 'Opened asset from position' in src/analytics/event.ts:175. The type argument only changes the properties, not the name.
  - `src/features/positions/stores/positionsStore.ts:167` (other): This is analytics.identify({...}) from the app's own wrapper in src/analytics/index.ts. It sets user properties (positionsAmount, positionsUSDValue, positionsAssetsAmount, positionsDappsAmount, positionsRewardsAmount, positionsRewardsUSDValue), once a day because of the throttle. It is not a vendor SDK call. The user_set tracker on analytics.identify covers it.
  - `src/features/positions/stores/trackPositionsResponseErrors.ts:43` (dynamic): The name is analytics.event.positionsError, the fixed string 'positions.error' (event.ts:176). The loop only changes the properties (kind price_not_found, skipped_portfolio_item or unknown).
  - `src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:41` (other): This is analytics.identify({stakedRnbw, rnbwMembershipTier}) from the app's wrapper. It sets user properties, not an event. The user_set tracker on analytics.identify covers it.
  - `src/features/rnbw-rewards/utils/claimAirdrop.ts:49` (dynamic): The name is analytics.event.rnbwAirdropClaimFailed, the fixed string 'rnbw_airdrop.claim.failed' (event.ts:357). It fires when preparing the claim fails (the catch block of prepareAirdropClaim).
  - `src/features/rnbw-rewards/utils/claimAirdrop.ts:91` (dynamic): The name is analytics.event.rnbwAirdropClaim, the fixed string 'rnbw_airdrop.claim' (event.ts:356). It fires after the claim status poll completes.
  - `src/features/rnbw-rewards/utils/claimAirdrop.ts:116` (dynamic): The name is analytics.event.rnbwAirdropClaimFailed, the fixed string 'rnbw_airdrop.claim.failed' (event.ts:357). It fires in the catch block of submitAirdropClaim.
  - `src/features/rnbw-rewards/utils/claimRewards.ts:75` (dynamic): The name is analytics.event.rnbwRewardsClaimFailed, the fixed string 'rnbw_rewards.claim.failed' (event.ts:353). It fires in the catch block of prepareRewardsClaim.
  - `src/features/rnbw-rewards/utils/claimRewards.ts:123` (dynamic): The name is analytics.event.rnbwRewardsClaim, the fixed string 'rnbw_rewards.claim' (event.ts:352). It fires after the claim status poll completes.
  - `src/features/rnbw-rewards/utils/claimRewards.ts:151` (dynamic): The name is analytics.event.rnbwRewardsClaimFailed, the fixed string 'rnbw_rewards.claim.failed' (event.ts:353). It fires in the catch block of submitRewardsClaim.
  - `src/features/rnbw-staking/utils/stakeRnbw.ts:194` (dynamic): trackStakeSuccess always sends analytics.event.rnbwStakingStake, the fixed string 'rnbw_staking.stake' (event.ts:360). It spreads the StakeSuccessMetadata object into the properties.
  - `src/features/rnbw-staking/utils/stakeRnbw.ts:213` (dynamic): trackStakeFailure always sends analytics.event.rnbwStakingStakeFailed, the fixed string 'rnbw_staking.stake.failed' (event.ts:361).
  - `src/features/rnbw-staking/utils/unstakeRnbw.ts:73` (dynamic): The name is analytics.event.rnbwStakingUnstakeFailed, the fixed string 'rnbw_staking.unstake.failed' (event.ts:363). It fires in the catch block when the unstake fails before it is submitted.
  - `src/features/rnbw-staking/utils/unstakeRnbw.ts:111` (dynamic): analytics.track(analytics.event.rnbwStakingUnstake, ...): the name is a key of the `event` constant map in src/analytics/event.ts:362 ('rnbw_staking.unstake'). Wrapper src/analytics/index.ts:102 forwards it to PostHog capture. This is the app's own wrapper, not a vendor call.
  - `src/features/rnbw-staking/utils/unstakeRnbw.ts:119` (dynamic): analytics.event.rnbwStakingUnstakeFailed resolves to a literal in src/analytics/event.ts:363. Fires in the catch of finalizeSubmittedUnstake.
  - `src/features/spot-pnl/ui/SpotPnlSheet.tsx:257` (dynamic): analytics.event.spotPnlSheetViewed is a literal in src/analytics/event.ts:334. Sent once per mount from a useEffect guarded by hasTrackedView.
  - `src/features/transfer/components/SendAssetFormField.js:71` (dynamic): analytics.event.sendMaxPressed is a literal in src/analytics/event.ts:93. Sent with no properties.
  - `src/features/transfer/screens/SendSheet.tsx:357` (dynamic): analytics.event.changedNativeCurrencyInputSend is a literal in src/analytics/event.ts:57. Sent with no properties.
  - `src/features/transfer/screens/SendSheet.tsx:378` (dynamic): analytics.event.changedTokenInputSend is a literal in src/analytics/event.ts:58. Sent with no properties.
  - `src/features/transfer/screens/SendSheet.tsx:554` (dynamic): analytics.event.resetAssetSelectionSend is a literal in src/analytics/event.ts:41. Sent with no properties.
  - `src/features/transfer/screens/SendSheet.tsx:670` (dynamic): analytics.event.tokenList is a literal in src/analytics/event.ts:225. Properties come from the `params` object: screen, no_icon, no_price, total_tokens.
  - `src/features/transfer/utils/trackSentTransaction.ts:32` (dynamic): analytics.event.sentTransaction is a literal in src/analytics/event.ts:59. trackSentTransaction is a second-level helper that builds the property object itself.
  - `src/features/wallet-connect/components/WalletConnectV2ListItem.tsx:112` (dynamic): analytics.event.manuallyDisconnectedFromWalletConnectConnection is a literal in src/analytics/event.ts:39. Fires after the 'disconnect' menu action completes.
  - `src/features/wallet-connect/handlers/onSessionProposal.ts:38` (dynamic): analytics.event.wcNewPairing is a literal in src/analytics/event.ts:164.
  - `src/features/wallet-connect/handlers/onSessionProposal.ts:62` (dynamic): analytics.event.wcNewSessionRejected is a literal in src/analytics/event.ts:167. Fires in rejectProposal after client.rejectSession.
  - `src/features/wallet-connect/handlers/onSessionProposal.ts:158` (dynamic): analytics.event.wcNewSessionApproved is a literal in src/analytics/event.ts:168. Fires after client.approveSession succeeds.
  - `src/features/wallet-connect/handlers/onSessionProposal.ts:175` (dynamic): analytics.track(analytics.event.wcRequestFailed, ...). The name is a member of the `event` constants object in src/analytics/event.ts (wcRequestFailed: 'wc.failed_request'), not a string literal. It is a single fixed value.
  - `src/features/wallet-connect/handlers/onSessionRequest.ts:84` (dynamic): analytics.event.wcRequestFailed, defined in src/analytics/event.ts:171 as 'wc.failed_request'.
  - `src/features/wallet-connect/handlers/onSessionRequest.ts:119` (dynamic): analytics.event.wcRequestFailed, defined in src/analytics/event.ts:171 as 'wc.failed_request'.
  - `src/features/wallet-connect/handlers/onSessionRequest.ts:147` (dynamic): analytics.event.wcRequestFailed, defined in src/analytics/event.ts:171 as 'wc.failed_request'.
  - `src/features/wallet-connect/handlers/onSessionRequest.ts:204` (dynamic): analytics.event.wcShowingSigningRequest, defined in src/analytics/event.ts:169 as 'Showing Walletconnect signing request'.
  - `src/features/wallet-connect/handlers/onSessionRequest.ts:214` (dynamic): analytics.event.wcRequestFailed, defined in src/analytics/event.ts:171 as 'wc.failed_request'. The call also sends method.
  - `src/handlers/deeplinks.ts:172` (dynamic): analytics.event.f2cProviderFlowCompleted, defined in src/analytics/event.ts:112 as 'f2c.provider.flow_completed'. This branch (Ramp) also sends success: true.
  - `src/handlers/deeplinks.ts:178` (dynamic): analytics.event.f2cProviderFlowCompleted, defined in src/analytics/event.ts:112 as 'f2c.provider.flow_completed'. This branch (other providers) omits success.
  - `src/hooks/useHiddenTokens.ts:34` (dynamic): analytics.event.toggledAnNFTAsHidden, defined in src/analytics/event.ts:50 as 'Toggled an NFT as Hidden'. The call sends isHidden: true.
  - `src/hooks/useHiddenTokens.ts:73` (dynamic): analytics.event.toggledAnNFTAsHidden, defined in src/analytics/event.ts:50 as 'Toggled an NFT as Hidden'. The call sends isHidden: false.
  - `src/hooks/useImportingWallet.ts:132` (dynamic): analytics.event.tappedImportButton, defined in src/analytics/event.ts:77 as 'Tapped "Import" button'. It is sent with no properties.
  - `src/hooks/useImportingWallet.ts:163` (dynamic): analytics.event.watchWallet, defined in src/analytics/event.ts:209 as 'watch_wallet'.
  - `src/hooks/useImportingWallet.ts:163` (dynamic): analytics.track(analytics.event.watchWallet, ...) goes through the wrapper in src/analytics/index.ts. The name is the member reference analytics.event.watchWallet, which src/analytics/event.ts:209 maps to 'watch_wallet'.
  - `src/hooks/useImportingWallet.ts:170` (dynamic): analytics.event.showWalletProfileModalForENSAddress, defined in src/analytics/event.ts:76
  - `src/hooks/useImportingWallet.ts:193` (dynamic): analytics.event.watchWallet, defined in src/analytics/event.ts:209
  - `src/hooks/useImportingWallet.ts:200` (dynamic): analytics.event.showWalletProfileModalForUnstoppableAddress, defined in src/analytics/event.ts:75
  - `src/hooks/useImportingWallet.ts:222` (dynamic): analytics.event.showWalletProfileModalForReadOnlyWallet, defined in src/analytics/event.ts:74
  - `src/hooks/useImportingWallet.ts:233` (dynamic): analytics.event.watchWallet, defined in src/analytics/event.ts:209
  - `src/hooks/useImportingWallet.ts:271` (dynamic): analytics.event.watchWallet, defined in src/analytics/event.ts:209
  - `src/hooks/useImportingWallet.ts:279` (dynamic): analytics.event.showWalletProfileModalForImportedWallet, defined in src/analytics/event.ts:73
  - `src/hooks/useImportingWallet.ts:337` (dynamic): analytics.event.importedSeedPhrase, defined in src/analytics/event.ts:72
  - `src/hooks/useIsOffline.ts:39` (dynamic): analytics.event.networkStatusReconnected, defined in src/analytics/event.ts:238
  - `src/hooks/useIsOffline.ts:47` (dynamic): analytics.event.networkStatusOffline, defined in src/analytics/event.ts:237. Fires after the debounce timer.
  - `src/hooks/useIsOffline.ts:52` (dynamic): analytics.event.networkStatusOffline, defined in src/analytics/event.ts:237. Fires immediately when debounceMs is 0.
  - `src/hooks/useOnAvatarPress.ts:111` (dynamic): analytics.event.viewedEnsProfile, defined in src/analytics/event.ts:51
  - `src/hooks/useRefreshAccountData.ts:59` (dynamic): analytics.track(analytics.event.refreshAccountData, { duration }). The name is a key of the `event` constant in src/analytics/event.ts, line 306, so it is a fixed literal.
  - `src/hooks/useScanner.ts:57` (dynamic): analytics.track(analytics.event.qrCodeScannedAddress). The name is a fixed literal in src/analytics/event.ts, line 280.
  - `src/hooks/useScanner.ts:78` (dynamic): analytics.track(analytics.event.qrCodeScannedProfile). The name is a fixed literal in src/analytics/event.ts, line 281.
  - `src/hooks/useScanner.ts:105` (dynamic): analytics.track(analytics.event.qrCodeScannedWalletConnect). The name is a fixed literal in src/analytics/event.ts, line 282.
  - `src/hooks/useScanner.ts:124` (dynamic): analytics.track(analytics.event.qrCodeScannedInvalid, { qrCodeData }). The name is a fixed literal in src/analytics/event.ts, line 283.
  - `src/hooks/useWalletCloudBackup.ts:56` (dynamic): analytics.track(analytics.event.iCloudNotEnabled, { category: 'backup' }). The name is a fixed literal in src/analytics/event.ts, line 71.
  - `src/hooks/useWalletCloudBackup.ts:66` (dynamic): analytics.track(analytics.event.viewHowToEnableICloud, { category: 'backup' }). The name is a fixed literal in src/analytics/event.ts, line 70.
  - `src/hooks/useWalletCloudBackup.ts:74` (dynamic): analytics.track(analytics.event.ignoreHowToEnableICloud, { category: 'backup' }). The name is a fixed literal in src/analytics/event.ts, line 69.
  - `src/hooks/useWalletCloudBackup.ts:131` (dynamic): The name is a ternary on cloudPlatform === 'Google Drive'. It picks errorDuringGoogleDriveBackup or errorDuringICloudBackup, and both are fixed literals in src/analytics/event.ts, lines 67-68. Properties are category, error and label.
  - `src/hooks/useWalletCloudBackup.ts:152` (dynamic): analytics.track(analytics.event.errorUpdatingBackupStatus, { category, label }). The name is a fixed literal in src/analytics/event.ts, line 66.
  - `src/hooks/useWalletSectionsData.ts:125` (dynamic): analytics.track(analytics.event.tokenList, params). Params are screen, no_icon, no_price and total_tokens, built in the effect above. The name is a fixed literal in src/analytics/event.ts, line 225.
  - `src/hooks/useWatchAssetUpdates.ts:70` (dynamic): analytics.track(event.minedTransactionAssetsTimedOut, { chainId, type }). `event` is imported from '@/analytics/event'. The name is a fixed literal in line 311 of that file.
  - `src/hooks/useWalletCohort.ts:64` (other): analytics.identify({...cohort counts}) goes through the Analytics wrapper in src/analytics/index.ts, line 77. It is not a direct SDK call. It sets traits such as ownedAccounts and watchedAccounts, and the distinct id is the deviceId, not a wallet address or email. The local `const identify` in the file only holds the counts.
  - `src/hooks/useWatchAssetUpdates.ts:173` (dynamic): The name is the `event` map constant `event.minedTransactionAssetsResolved`, defined in src/analytics/event.ts:310 as a plain string. It's one fixed name, written as a map property rather than a literal.
  - `src/hooks/useWatchPendingTxs.ts:92` (dynamic): The name is `event.pendingTransactionResolved`, defined in src/analytics/event.ts:309. It's one fixed name.
  - `src/model/wallet.ts:450` (dynamic): The name is `analytics.event.walletCreateFailed`. The Analytics class re-exposes the `event` map (`event = event` in src/analytics/index.ts:31), and the value is 'wallet_create.failed' (src/analytics/event.ts:242).
  - `src/model/wallet.ts:498` (dynamic): The name is `analytics.event.walletCreateFailed`, a fixed name. Only the `error` property varies ('wallet_already_exists').
  - `src/model/wallet.ts:506` (dynamic): The name is `analytics.event.walletCreateFailed`, a fixed name. The `error` property is 'wallet_already_exists_restoring'.
  - `src/model/wallet.ts:722` (dynamic): The name is `analytics.event.walletCreateSucceeded`, which is 'wallet_create.succeeded' (src/analytics/event.ts:241). It's sent with the analyticsParams object built earlier in createWallet.
  - `src/model/wallet.ts:730` (dynamic): The name is `analytics.event.walletCreateFailed`, a fixed name. It's sent from the catch block with `error` set to error.message.
  - `src/notifications/analytics.ts:11` (dynamic): The name is `analytics.event.notificationsPromoTapped`, defined in src/analytics/event.ts:101. It's one fixed name.
  - `src/notifications/analytics.ts:17` (dynamic): The name is `analytics.event.notificationsPromoNotificationSettingsChanged`, defined in src/analytics/event.ts:100. It's one fixed name.
  - `src/performance/tracking/index.ts:86` (dynamic): trackMetric(metric, ...) forwards a name its callers choose. The parameter type PerformanceEvent (line 12) is only `event.performanceReport | event.performanceInitializeWallet`, so the name is always one of those two values (src/analytics/event.ts:246-247).
  - `src/redux/settings.ts:121` (dynamic): The name is `analytics.event.debugIdentifyProbe`, defined in src/analytics/event.ts:85. It's a temporary probe (FEPLAT-67), as the code comment says. The paired probe is in src/App.tsx:177.
  - `src/resources/addys/claimables/query.ts:83` (other): analytics.identify({ claimablesAmount, claimablesUSDValue, ...claimable-<type>-USDValue }) through the '@/analytics' wrapper, throttled to once an hour. The spread adds trait names built at runtime from claimable.type (claimable-<type>-USDValue). It is not a direct SDK call.
  - `src/resources/favorites.ts:178` (dynamic): analytics.track(analytics.event.addFavoriteToken, ...) passes a key of the `event` map in src/analytics/event.ts:208 instead of a string literal. The map value is 'add_favorite_token'. The call goes through the '@/analytics' wrapper, not a vendor SDK.
  - `src/screens/AddWalletSheet.tsx:45` (dynamic): analytics.event.addWalletFlowStarted resolves to 'add_wallet_flow.started' (src/analytics/event.ts:92). The call goes through the '@/analytics' wrapper. It is sent with type 'new'.
  - `src/screens/AddWalletSheet.tsx:49` (dynamic): analytics.event.tappedCreateNewWallet resolves to 'Tapped Create New Wallet' (src/analytics/event.ts:46), sent with no properties.
  - `src/screens/AddWalletSheet.tsx:58` (dynamic): analytics.event.tappedAddExistingWallet resolves to 'Tapped Add Existing Wallet' (src/analytics/event.ts:45), sent with no properties.
  - `src/screens/AddWalletSheet.tsx:59` (dynamic): analytics.event.addWalletFlowStarted resolves to 'add_wallet_flow.started' (src/analytics/event.ts:92). It is sent with type 'seed'.
  - `src/screens/AddWalletSheet.tsx:70` (dynamic): analytics.event.tappedWatchAddress resolves to 'Tapped Watch Address' (src/analytics/event.ts:49), sent with no properties.
  - `src/screens/AddWalletSheet.tsx:71` (dynamic): analytics.event.addWalletFlowStarted resolves to 'add_wallet_flow.started' (src/analytics/event.ts:92). It is sent with type 'watch'.
  - `src/screens/AddWalletSheet.tsx:82` (dynamic): analytics.event.addWalletFlowStarted resolves to 'add_wallet_flow.started' (src/analytics/event.ts:92). It is sent with type 'seed' from the restore-from-cloud button.
  - `src/screens/AddWalletSheet.tsx:98` (dynamic): analytics.event.addWalletFlowStarted resolves to 'add_wallet_flow.started' (src/analytics/event.ts:92). It is sent with type 'ledger_nano_x' and isFirstWallet false from the hardware wallet button.
  - `src/screens/Airdrops/utils.ts:121` (dynamic): analytics.event.claimClaimableSucceeded resolves to 'claim_claimable.succeeded' (src/analytics/event.ts:212). It fires after provider.sendTransaction succeeds in claimAirdrop.
  - `src/screens/Airdrops/utils.ts:150` (dynamic): analytics.event.claimClaimableFailed resolves to 'claim_claimable.failed' (src/analytics/event.ts:213). It fires in the catch block of claimAirdrop.
  - `src/screens/Airdrops/utils.ts:150` (dynamic): analytics.track(analytics.event.claimClaimableFailed, ...) goes through the wrapper in src/analytics/index.ts. The name is a lookup into the event map in src/analytics/event.ts:213, so it is not a literal at the call site. It always sends 'claim_claimable.failed'.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:264` (dynamic): analytics.event.tappedDoneEditingWallet resolves to a string in src/analytics/event.ts:61, so the name is fixed.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:267` (dynamic): analytics.event.tappedCancelEditingWallet resolves to a string in src/analytics/event.ts:62.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:286` (dynamic): analytics.event.tappedEditWallet resolves to a string in src/analytics/event.ts:63.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:295` (dynamic): analytics.event.tappedNotificationSettings resolves to a string in src/analytics/event.ts:64.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:317` (dynamic): analytics.event.tappedDeleteWallet resolves to a string in src/analytics/event.ts:47.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:341` (dynamic): analytics.event.tappedDeleteWalletConfirm resolves to a string in src/analytics/event.ts:65.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:397` (dynamic): analytics.event.pressedButton resolves to a string in src/analytics/event.ts:81. The call passes buttonName and action.
  - `src/screens/change-wallet/ChangeWalletSheet.tsx:411` (dynamic): analytics.event.tappedEdit resolves to a string in src/analytics/event.ts:48.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:74` (dynamic): Failure of the GET claim request. analytics.event.claimClaimableFailed resolves to a fixed string in src/analytics/event.ts:213.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:95` (dynamic): Failure of the POST claim request. Same fixed name from src/analytics/event.ts:213.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:115` (dynamic): The API answered with payload.success false. Same fixed name from src/analytics/event.ts:213.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:136` (dynamic): The claim succeeded. analytics.event.claimClaimableSucceeded resolves to a fixed string in src/analytics/event.ts:212.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:157` (dynamic): analytics.track(analytics.event.claimClaimableFailed, ...) is the app's own wrapper (src/analytics/index.ts, PostHog behind it), not a vendor SDK. The scanner can't follow the name through the `analytics.event` property. src/analytics/event.ts:213 maps it to 'claim_claimable.failed'.
  - `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:175` (dynamic): Same wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent from onSuccess when the claim status is still 'claiming'.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:417` (dynamic): Wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent when the rap fails at the claim step.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:440` (dynamic): Wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent when the rap fails at the swap/bridge step.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:469` (dynamic): Wrapper call with analytics.event.claimClaimableSucceeded, which event.ts:212 maps to 'claim_claimable.succeeded'.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:491` (dynamic): Wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent from the catch block.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:515` (dynamic): Wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent from onError.
  - `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:534` (dynamic): Wrapper call with analytics.event.claimClaimableFailed (event.ts:213), sent from onSuccess when the status is still 'claiming'.
  - `src/screens/expandedAssetSheet/components/sections/BalanceSection.tsx:61` (dynamic): Wrapper call with analytics.event.spotPnlPillShown, which event.ts:335 maps to 'spot_pnl.pill.shown'. A ref makes it send once.
  - `src/screens/expandedAssetSheet/components/sections/TopHoldersSection.tsx:44` (dynamic): Wrapper call with analytics.event.topHoldersShown, which event.ts:338 maps to 'top_holders.shown'. A ref makes it send once, in an effect.
  - `src/screens/FiatOnRamp/ProviderListItem.tsx:35` (dynamic): Wrapper call with analytics.event.f2cProviderFlowStarted, which event.ts:107 maps to 'f2c.provider.flow_opened'.
  - `src/screens/LearnWebViewScreen.tsx:35` (dynamic): Wrapper call with analytics.event.learnArticleOpened, which event.ts:88 maps to 'learn_article.opened'. It is sent from the effect cleanup when the screen unmounts.
  - `src/screens/LearnWebViewScreen.tsx:51` (dynamic): Wrapper call with analytics.event.learnArticleShared, which event.ts:89 maps to 'learn_article.shared'.
  - `src/screens/LearnWebViewScreen.tsx:51` (dynamic): Name is the key analytics.event.learnArticleShared, a constant in the `event` map in src/analytics/event.ts (`event = event` on the Analytics class). The call goes through the wrapper analytics.track; the scanner doesn't resolve the member lookup.
  - `src/screens/NoNeedWCSheet.tsx:36` (dynamic): analytics.event.wcRequestFailed resolves to a fixed string in src/analytics/event.ts:171. Wrapper call on mount.
  - `src/screens/ProfileSheet.tsx:68` (dynamic): analytics.event.viewedProfile resolves to a fixed string in src/analytics/event.ts:52.
  - `src/screens/rewards/components/RewardsClaimed.tsx:87` (dynamic): analytics.event.rewardsPressedAvailableCard resolves to a fixed string in src/analytics/event.ts:158.
  - `src/screens/rewards/components/RewardsEarnings.tsx:71` (dynamic): analytics.event.rewardsPressedPendingEarningsCard resolves to a fixed string in src/analytics/event.ts:157.
  - `src/screens/rewards/components/RewardsLeaderboardItem.tsx:81` (dynamic): analytics.event.rewardsPressedLeaderboardItem resolves to a fixed string in src/analytics/event.ts:162.
  - `src/screens/rewards/components/RewardsStats.tsx:34` (dynamic): analytics.event.rewardsPressedBridgedCard resolves to a fixed string in src/analytics/event.ts:161.
  - `src/screens/rewards/components/RewardsStats.tsx:42` (dynamic): analytics.event.rewardsPressedSwappedCard resolves to a fixed string in src/analytics/event.ts:160.
  - `src/screens/rewards/RewardsSheet.tsx:45` (dynamic): analytics.event.rewardsViewedSheet resolves to a fixed string in src/analytics/event.ts:125.
  - `src/screens/SettingsSheet/components/Backups/ShowSecretView.tsx:10` (dynamic): analytics.event.showSecretView resolves to a fixed string in src/analytics/event.ts:43.
  - `src/screens/SettingsSheet/components/Backups/ViewWalletBackup.tsx:140` (dynamic): analytics.event.addWalletFlowStarted resolves to a fixed string in src/analytics/event.ts:92.
  - `src/screens/SettingsSheet/components/CurrencySection.tsx:40` (dynamic): analytics.event.changedNativeCurrency resolves to a fixed string in src/analytics/event.ts:55.
  - `src/screens/SettingsSheet/components/LanguageSection.tsx:24` (dynamic): analytics.event.changedLanguage resolves to a fixed string in src/analytics/event.ts:54.
  - `src/screens/SettingsSheet/components/NetworkSection.tsx:30` (dynamic): analytics.track(analytics.event.changedNetwork, ...) goes through the wrapper in src/analytics/index.ts. The name is a key of the `event` object in src/analytics/event.ts:56, which maps to a literal string, so one name is possible.
  - `src/screens/SettingsSheet/components/PrivacySection.tsx:35` (dynamic): analytics.event.analyticsTrackingDisabled resolves to a literal in src/analytics/event.ts:83. It is sent just before analytics.disable().
  - `src/screens/SettingsSheet/components/PrivacySection.tsx:42` (dynamic): analytics.event.analyticsTrackingEnabled resolves to a literal in src/analytics/event.ts:84. It is sent right after analytics.enable().
  - `src/screens/token-launcher/components/TokenLauncherFooter.tsx:61` (dynamic): analytics.event.tokenLauncherWalletLoadFailed resolves to a literal in src/analytics/event.ts:234. It fires when the wallet is a hardware wallet.
  - `src/screens/token-launcher/components/TokenLauncherFooter.tsx:79` (dynamic): Same event key (event.ts:234). It fires when the private key is null or the user is not authenticated.
  - `src/screens/token-launcher/components/TokenLauncherFooter.tsx:120` (dynamic): Same event key (event.ts:234). It fires in the catch block when loading the wallet throws.
  - `src/screens/token-launcher/components/TokenLauncherFooter.tsx:212` (dynamic): analytics.event.tokenLauncherSharePressed resolves to a literal in src/analytics/event.ts:230. It fires after Share.share returns.
  - `src/screens/token-launcher/components/TokenLauncherHeader.tsx:59` (dynamic): analytics.event.tokenLauncherAbandoned resolves to a literal in src/analytics/event.ts:231. It fires when the user confirms discard and exit in the action sheet.
  - `src/screens/token-launcher/hooks/useUploadToCloudinary.ts:29` (dynamic): analytics.event.tokenLauncherImageUploadFailed resolves to a literal in src/analytics/event.ts:233. It fires in the catch block of the upload.
  - `src/screens/token-launcher/state/tokenLauncherStore.ts:292` (dynamic): analytics.event.tokenLauncherStepChanged resolves to a literal in src/analytics/event.ts:228. It fires in setStep.
  - `src/screens/token-launcher/state/tokenLauncherStore.ts:379` (dynamic): analytics.event.tokenLauncherTokenCreated resolves to a literal in src/analytics/event.ts:229. It fires after launchToken succeeds.
  - `src/screens/token-launcher/state/tokenLauncherStore.ts:438` (dynamic): analytics.event.tokenLauncherCreationFailed resolves to a literal in src/analytics/event.ts:232. It fires in the catch block of the launch.
  - `src/screens/WalletConnectApprovalSheet.tsx:260` (dynamic): analytics.event.shownWalletconnectSessionRequest resolves to a literal in src/analytics/event.ts:44. It fires on mount, after interactions.
  - `src/screens/WalletConnectApprovalSheet.tsx:260` (dynamic): analytics.track(analytics.event.shownWalletconnectSessionRequest): the name is a key of the const `event` map in src/analytics/event.ts (line 44), not a literal. Always this one name.
  - `src/screens/WalletConnectApprovalSheet.tsx:298` (dynamic): analytics.track(analytics.event.receivedWcConnection, {dappName, dappUrl, waitingTime}): name comes from the `event` map (event.ts:40).
  - `src/screens/WelcomeScreen/WelcomeScreen.tsx:163` (dynamic): analytics.track(analytics.event.welcomeNewWallet) on the create-wallet press; name from the `event` map (event.ts:296).
  - `src/screens/WelcomeScreen/WelcomeScreen.tsx:201` (dynamic): analytics.track(analytics.event.welcomeAlreadyHave) when the restore sheet opens; name from the `event` map (event.ts:297).
  - `src/state/performance/performance.ts:62` (dynamic): analytics.track(analytics.event.performanceTimeToSignOperation, log): fixed key of the `event` map (event.ts:206); the whole PerformanceLog object is sent as properties.
  - `src/state/performance/performance.ts:69` (dynamic): analytics.track(analytics.event.performanceTimeToSign, {screen, completedAt, elapsedTime, totalElapsedTime, error}) when isEndOfFlow; name from the `event` map (event.ts:205).
  - `src/state/wallets/initializeWallet.ts:190` (dynamic): analytics.track(event.walletInitializationFailed, {error, walletStatus}) in the catch block; `event` is imported from '@/analytics/event' (event.ts:243).
  - `src/utils/reviewAlert.ts:108` (dynamic): analytics.track(event.appStoreReviewPrompted, {action, promptCount}) after StoreReview.requestReview(); `event` imported from '@/analytics/event' (event.ts:303).
  - `src/state/wallets/initializeWallet.ts:135` (wrapper): analytics.identify() goes through the src/analytics/index.ts wrapper (the PostHog identify with the device id), not a vendor SDK. Already listed as an identify tracker; the vendor flag comes from the generic `analytics.` object name.
  - `src/utils/branch.ts:127` (wrapper): analytics.identify({branchCampaign, branchReferrer, branchReferringLink}) imports `analytics` from '@/analytics', the wrapper (index.ts:77). It sets person properties through the wrapper, not a vendor SDK.
  - `src/analytics/index.test.ts:72` (not_analytics): Unit test of the Analytics wrapper (mocked PostHog client); the argument is analytics.event.pressedButton, a fixed name. Nothing is sent.
  - `src/analytics/index.test.ts:86` (not_analytics): Unit test of the wrapper; passes analytics.event.perpsWithdrew with a mocked client. Not product tracking.
  - `src/analytics/index.test.ts:101` (not_analytics): Unit test of the wrapper (AppsFlyer UID is added to the event); mocked client, not product tracking.
  - `src/analytics/index.test.ts:131` (not_analytics): Unit test of analytics.screen on a local Analytics instance with a mocked client.
  - `src/analytics/index.test.ts:146` (not_analytics): Unit test that tracking is dropped after disable(); mocked client.
  - `src/analytics/index.test.ts:148` (not_analytics): Unit test that screen is dropped after disable(); mocked client.
  - `src/analytics/index.test.ts:179` (not_analytics): Unit test of the startup queue; mocked client.
  - `src/analytics/index.test.ts:199` (not_analytics): Unit test of disabling during initialization; mocked client.
  - `src/analytics/index.test.ts:233` (not_analytics): Unit test with PostHog config missing; mocked client.
  - `src/analytics/index.test.ts:234` (not_analytics): Unit test with PostHog config missing; mocked client.
  - `src/navigation/onNavigationStateChange.js:130` (dynamic): analytics.screen(routeName, paramsToTrack) sends a screen event for every navigation change. The name is the active route name from the navigation state, one of the Routes values in src/navigation/routesNames. It comes from navigation data, so no fixed list. Properties come from getScreenTrackingParams, which returns asset details for the expanded asset sheets.
  - ios/AppDelegate.swift reports to Sentry (SentrySDK.capture) for sandbox-escape warnings and applicationWillTerminate while a RAP is running. This is error monitoring, not product analytics, and no tracker was added for it.
  - The only SDK is posthog-react-native 4.77.0 (package.json), used only inside src/analytics/index.ts. The app is React Native, so everything runs on the client and there is no server code.
  - Analytics.track adds walletAddressHash and walletType to every event, taken from setWalletContext. On Android it also adds device_brand, device_manufacturer and device_model. When AppsFlyer has a uid it adds appsflyer_id.
  - src/analytics/index.ts sends 'Application Updated' (previous_version, previous_build) on its own at init when the version changed but the build number didn't. The PostHog client also sends its own autocaptured events (e.g. 'Application Installed'). before_send drops 'Application Installed' for returning users.
  - Analytics.screen(route, params) sends PostHog's $screen event. It is called from src/navigation/onNavigationStateChange.js with the route name. It has no tracker because it doesn't send named events.
  - Analytics.identify uses the device id as the PostHog distinct id, set at init via analytics.init({deviceId}), with the traits as person properties. There is no reset or alias call anywhere in the wrapper. The PostHog client is bootstrapped with distinctId=deviceId.
  - Tracking is disabled in tests (IS_TEST) and when the user sets doNotTrack. Calls made before init are queued and flushed once PostHog is ready.
  - Callers that forward a name: src/performance/tracking/index.ts:86 (analytics.track(metric, ...)), src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts (a local alias of analytics.track), src/features/placements/engagement/trackInteraction.ts, src/features/discover/utils/trackDiscoverCardPress.ts and src/__swaps__/utils/trackSwapEvent.ts. Expect these in the runtime-names list in round two.
  - src/__swaps__/utils/trackSwapEvent.ts builds the full swap property set itself (asset address, chain, name, symbol, amounts, isBridge, degenMode, isHardwareWallet, tradeAmountUSD, errorMessage) and sends it on both swaps.submitted and swaps.failed. errorMessage is null on submitted.
  - The scanner flagged only the 11 calls under src/__swaps__/ as unreadable, although the same analytics.track(analytics.event.X) form is used in about 277 other places that it reads without complaint. I could not tell why the swaps files differ, so I credited their names through 'explained'. Check the scan again after this round.
  - src/analytics/index.ts (the wrapper) merges wallet context into each event and adds attribution properties such as appsflyer_id (see src/analytics/index.test.ts:95-106).
  - Every call site passes the event name as analytics.event.<key>. The `event` object in src/analytics/event.ts maps each key to the real name (for example pressedButton is 'Pressed Button'). Names are literals in that one file, so the scanner can't read them at the call site. Each flagged call needs an `explained` entry with names taken from event.ts. The 'event name built at runtime' lines are not data-driven.
  - The '@/analytics' wrapper (src/analytics/index.ts) sends through PostHog (posthog-react-native), not a vendor SDK that the scanner should flag. The 'vendor SDK called outside a wrapper' lines on analytics.track and analytics.identify come from the shared `analytics` object name. src/analytics/index.ts is already a wrapper.
  - src/analytics/index.ts adds walletAddressHash and walletType to every track, identify and screen call. On Android it also adds device_brand, device_manufacturer and device_model. When AppsFlyer has a uid, track also adds appsflyer_id.
  - The wrapper sends 'Application Updated' itself (captureVersionUpdate, src/analytics/index.ts:169) with previous_version and previous_build. posthog-react-native also sends lifecycle events on its own: Application Installed, Opened, Became Active and Updated. before_send drops Application Installed for returning users and drops everything while analytics is disabled.
  - analytics.screen(route, params) in src/analytics/index.ts:91 sends a PostHog screen event. It is called from src/navigation/onNavigationStateChange.js:130 with a route name from navigation state, so the name comes from data.
  - Analytics is switched off in tests (IS_TEST) and when the user sets doNotTrack. Calls are queued until init({ deviceId }) has run. Identity uses the device id, not a wallet or user id.
  - src/analytics/index.test.ts calls analytics.track and analytics.identify on a locally built Analytics instance with a mocked client. It is covered by a direct_sdk_exceptions entry.
  - The 'vendor SDK called outside a wrapper' lines are not SDK calls. They are analytics.track(...) on the app's own Analytics class (src/analytics/index.ts), imported from '@/analytics'. The only direct PostHog calls are inside that wrapper file. The scanner flags them only because `analytics.track` looks like a vendor shape and the name can't be read.
  - Every call reads its name as `analytics.event.<key>` or `event.<key>`. Both point to the `event` object in src/analytics/event.ts (key to string-literal value), so the scanner can't read them as literals. The event.ts file also holds the typed EventProperties map: `[event.<key>]: {...}` per event. That map is the best source for each event's properties when completing the registry.
  - src/analytics/index.ts sends on its own: client.capture('Application Updated', { previous_version, previous_build }) from captureVersionUpdate; analytics.screen(route, params) calls PostHog screen; the PostHog SDK adds its own lifecycle events such as Application Installed (filtered in before_send for returning users).
  - Analytics.track adds appsflyer_id when AppsFlyer has a uid, plus device_brand, device_manufacturer and device_model on Android. walletAddressHash and walletType are added on every call.
  - Analytics is disabled when IS_TEST is set or device doNotTrack is set. PostHog is configured with disableSurveys and disableRemoteFeatureFlags, and the distinct id is the device id (bootstrap with isIdentifiedId true).
  - Event names across the app are not string literals at the call site. Callers write `analytics.event.<key>` or `event.<key>`. Both point at the `event` constant map in src/analytics/event.ts, which maps each key to its fixed string (for example `setAppIcon: 'Set App Icon'`). Property types are in the `EventProperties` type in the same file. The same shape is likely behind the rest of the unreadable lines, beyond the 25 listed.
  - src/analytics/index.ts `track` adds `walletAddressHash` and `walletType` to every event. On Android it also adds `device_brand`, `device_manufacturer` and `device_model`, and it adds `appsflyer_id` when AppsFlyer has a uid.
  - src/analytics/index.ts `identify` uses the device id (`deviceId`) as the PostHog distinct id, and merges the wallet metadata into the traits. `analytics.screen(route, params)` also sends a PostHog `screen` event.
  - src/analytics/index.ts sends an 'Application Updated' event itself (captureVersionUpdate, line 169). It drops the first 'Application Installed' for returning users in the `before_send` hook (line 200).
  - Every call in this app is analytics.track(analytics.event.<key>, props). `event` in src/analytics/event.ts maps each key to the real name (e.g. backupChoosePassword -> 'Choose Password Step', cashAmountEntered -> 'cash.amount_entered'), and the registry should hold these values. There are about 305 such calls in 145 files, and the scanner can't read the member path analytics.event.<key>. That is why the list keeps showing them and why it finds 0 events. The lines above are only the first batch; the rest follow the same pattern. If the scanner can't resolve analytics.event.<key> from src/analytics/event.ts, every other call site will need the same `explained` entry.
  - src/analytics/index.ts is a PostHog wrapper (posthog-react-native) and sends some things itself. track() always merges walletAddressHash and walletType, plus appsflyer_id when AppsFlyer has a uid, plus device_brand, device_manufacturer and device_model on Android. captureVersionUpdate() sends 'Application Updated' with previous_version and previous_build when the version changed but the build number stayed the same. PostHog's own autocaptured events such as 'Application Installed' pass through before_send, which drops 'Application Installed' for returning users.
  - analytics.screen(route, params, walletContext) in src/analytics/index.ts:91 sends a PostHog screen event named by a Routes constant. The scanner doesn't read it as a tracker yet. Its callers weren't checked in this round.
  - The wrapper is disabled when IS_TEST is set or the user has the doNotTrack device setting. It then sends nothing.
  - Every flagged line calls analytics.track(analytics.event.<key>, ...) on the wrapper in src/analytics/index.ts. `analytics.event` is the `event` map from src/analytics/event.ts, which maps camelCase keys to dotted strings such as 'cash.phone_submitted'. The names are fixed constants, not runtime data. The 'vendor SDK called outside a wrapper' lines are not SDK calls. The wrapper's own track() calls posthog `client.capture` in src/analytics/index.ts:107 and is already listed in `wrappers`.
  - The `names` values are the strings from src/analytics/event.ts, which is also where each event's properties are typed (the EventProperties map near line 573 onward).
  - The wrapper's track() merges walletContext, metadata and attribution into every capture (src/analytics/index.ts:107). Its `before_send` hook drops 'Application Installed' for returning users (line 200).
  - The scanner reports 0 events because most callers pass analytics.event.<key> rather than a string literal. This probably affects far more than the 13 lines flagged here, so check the other callers of analytics.track in the next step.
  - Every flagged line is analytics.track(analytics.event.<key>) through the wrapper in src/analytics/index.ts. The names are keys of the `event` const map in src/analytics/event.ts, and each key has one string value, so no name is built at runtime. They look unreadable because the scanner can't follow `analytics.event.<key>` to the map.
  - Only the cash.kyc_*, cash.sign_in_* and cash.card_link* keys are flagged. Several *.test.ts files (cashSignInService.test.ts, cardLinkFlowStore.test.ts, useSubmitReviewFlow.test.ts, verifyPhoneFlowStore.test.ts, kycReturnFlowStore.test.ts) mock `event` with copies of these keys, which may be what makes them ambiguous to the scanner. I did not confirm this.
  - The same analytics.track(analytics.event.<key>) shape is used in about 145 files (305 calls), such as src/screens/ProfileSheet.tsx:68. Those may also read as unresolved names once they are scanned, so if they are flagged, use the event map in src/analytics/event.ts to resolve them in the same way.
  - src/analytics/index.ts (the wrapper) sends things on its own that are not in the registry from callers. It calls PostHog capture('Application Updated') on a version change with the same build number (captureVersionUpdate). It captures an 'Application Installed' event through the PostHog SDK's own autocapture, and drops that event for returning users in before_send. It also sends client.screen(route, ...) from analytics.screen(), and it merges walletAddressHash, walletType, Android device_brand, device_manufacturer and device_model, plus appsflyer_id when available, into every track() call.
  - Every call in the app reads analytics.track(analytics.event.<key>, props). `analytics.event` is the `event` constant in src/analytics/event.ts (key -> string value), with property types in `EventProperties` in the same file. The name is always a fixed string, but written as a member expression, so the scanner can't read it. The explained entries give the real values. Other call sites of the same shape will be flagged the same way. The fix is to resolve `analytics.event.<key>` through src/analytics/event.ts. If the scanner can't, add each one to `explained` with its value from that map.
  - src/analytics/index.ts wraps posthog-react-native (PostHog.capture, identify, screen) and AppsFlyer (src/analytics/appsflyer.ts). It adds walletAddressHash and walletType, android device_brand / device_manufacturer / device_model, and appsflyer_id when known, to every track call. This is the only file that calls PostHog.
  - src/analytics/index.ts sends 'Application Updated' (previous_version, previous_build) on its own, from captureVersionUpdate, when the version changes but the build number is the same. PostHog also sends its own lifecycle events ('Application Installed', filtered out for returning users in before_send).
  - analytics.screen(route, params) sends PostHog screen events from src/navigation/onNavigationStateChange.js:130. The name is the route name at runtime, so it is not a literal event. Screen names come from the Routes constants (src/navigation/routesNames).
  - analytics.disable() / enable() / the device.doNotTrack setting turns all tracking and AppsFlyer off in src/analytics/index.ts. Tracking is also off in tests (IS_TEST). Identify uses the deviceId as the distinct id, not a wallet or user id.
  - Events are queued in src/analytics/index.ts until PostHog is ready, so early events are delayed rather than dropped. They are lost if init is never called or POSTHOG_API_KEY / POSTHOG_HOST are missing.
  - Every analytics.track call in this repo (about 300 calls across 142 files) passes its name as a key of the `event` const object in src/analytics/event.ts, written as analytics.event.X, event.X or analyticsEvent.X. I found no string-literal names. The scanner doesn't resolve those keys, so the 12 lines above are only the first batch of a longer list, and the rest need the same kind of mapping.
  - The mapping is the `event` object in src/analytics/event.ts (lines 38-364, `as const`). Per-event property types are in the EventProperties map in the same file, keyed `[event.X]`.
  - src/analytics/index.ts (PostHog via posthog-react-native) sends these on its own: `screen` (analytics.screen(route, params)) and an 'Application Updated' capture from captureVersionUpdate. The PostHog SDK also sends its own 'Application Installed', which before_send drops for returning users. Every track and screen call gets walletAddressHash, walletType and appsflyer_id (when set). Android calls also get device_brand, device_manufacturer and device_model.
  - analytics.identify(userProperties) in the wrapper identifies with the device id (not a wallet address or email) and merges walletAddressHash and walletType into the traits. The wrapper has no reset or alias method.
  - Analytics is disabled when IS_TEST is set or device 'doNotTrack' is true, and it can be turned on and off at runtime with enable() and disable().
  - The SDK is posthog-react-native, called only inside src/analytics/index.ts. The wrapper's own `analytics.track` is a typed method, so calls pass `analytics.event.X`, not a string.
  - Every analytics.track call in the app reads its name as analytics.event.<key>. The keys map to the real event names in src/analytics/event.ts, and the allowed properties per event are typed in EventProperties in the same file. The scanner doesn't follow that member access, so the calls flagged above look runtime-built and 'outside a wrapper'. They all import { analytics } from '@/analytics' and go through the wrapper in src/analytics/index.ts. Other call sites with the same shape will be flagged too, and the real names are the string values in event.ts.
  - src/analytics/index.ts (PostHog via posthog-react-native) adds walletAddressHash and walletType to every event and identify. On Android it also adds device_brand, device_manufacturer and device_model. It adds appsflyer_id to events when the AppsFlyer uid exists.
  - src/analytics/index.ts sends 'Application Updated' (previous_version, previous_build) on its own, from captureVersionUpdate, when the version changes and the build number stays the same. The PostHog client also captures its own lifecycle events, such as 'Application Installed', which before_send drops for returning users.
  - The wrapper's disabled flag (IS_TEST or the doNotTrack device setting) turns off all events and identify calls.
  - analytics.screen(route, params) sends a PostHog screen event. It appears only in src/analytics/README.md and the tests, with no product callers.
  - Every app call reads analytics.track(analytics.event.<key>, props) or analytics.track(event.<key>, props). The name is a constant in the `event` object in src/analytics/event.ts, not a string literal, and about 305 calls in 145 files have this shape. The scanner cannot resolve that reference by itself, so the list is probably a sample of a wider pattern. If more `analytics.event.*` or `event.*` lines come back, they resolve the same way: look the key up in src/analytics/event.ts.
  - src/analytics/index.ts is a PostHog (posthog-react-native) wrapper class, exported as the instance `analytics`. track() adds walletAddressHash and walletType from DefaultMetadata. On Android it also adds device_brand, device_manufacturer and device_model. When an AppsFlyer uid exists, it adds appsflyer_id. Its own sends: captureVersionUpdate() sends 'Application Updated' with previous_version and previous_build, and screen() sends a PostHog screen event for a route.
  - src/analytics/index.ts before_send drops 'Application Installed' for returning users (device isReturningUser). PostHog also sends its own lifecycle events such as 'Application Installed', and those are not in the code.
  - identify() in src/analytics/index.ts always uses the device id (deviceId) as the distinct id. The traits passed in are merged with wallet metadata. No reset or alias call exists anywhere in the scan.
  - The flagged files also use trackSwapEvent and analytics.screen. analytics.screen has no tracker yet. I did not check whether any caller uses it.
  - The same cause sits behind all the 'runtime name' and 'vendor SDK outside a wrapper' lines. The code calls analytics.track(analytics.event.<key>) or analytics.track(event.<key>), where `event` is a `const ... as const` object in src/analytics/event.ts (about 330 entries, key -> string). The scanner can't resolve that object, because Analytics also re-exposes it as the class field `event = event`. These lines are not new SDK calls: they all go through the `analytics` wrapper. The same shape is used in about 305 calls across 145 files, so expect the same lines for the rest of the codebase. Fix it by reading the key -> value map in src/analytics/event.ts for the names. Don't treat them as blind spots.
  - src/analytics/index.ts (wrapper, client) sends several things itself. The Analytics.screen() method sends PostHog screen events using the route name. captureVersionUpdate() sends 'Application Updated' with previous_version and previous_build. The PostHog client also auto-captures 'Application Installed', which before_send drops for returning users.
  - Analytics.track() always adds walletAddressHash and walletType, plus appsflyer_id when AppsFlyer has a uid. On Android it also adds device_brand, device_manufacturer and device_model. Callers can override the wallet context through the third argument (src/analytics/index.ts:102-108).
  - Identity: Analytics.identify() always uses the deviceId as the distinct id and merges the traits with the wallet metadata. No reset or alias exists in the wrapper. The scanner sees many identify calls because userProperties set through analytics.identify() are the app's user-property mechanism (src/analytics/userProperties.ts).
  - Every event name in the app is a key in the `event` const object in src/analytics/event.ts, which is `as const` and has a typed `EventProperties` map. Calls look like `analytics.track(analytics.event.<key>, props)`. About 296 `analytics.event.` uses sit in 128 files. The `event` field on the Analytics class (src/analytics/index.ts:31) is an alias of that object. The names are never built from data. If the scanner can't resolve the member access, the cause is the same at every site, not a per-call problem.
  - src/analytics/index.ts wraps posthog-react-native (PostHog). Its `track` adds walletAddressHash, walletType, appsflyer_id (when AppsFlyer has a uid), and on Android device_brand, device_manufacturer and device_model.
  - src/analytics/index.ts:169 sends 'Application Updated' (previous_version, previous_build) on its own, from captureVersionUpdate when the version changes but the build number stays the same. PostHog also captures its own lifecycle events such as 'Application Installed' (src/analytics/index.ts:200). The first launch is dropped for returning users.
  - src/analytics/index.ts `screen(route, params)` sends PostHog screen events named by the route. `identify` uses the deviceId as the distinct id, not a wallet address, and attaches the hashed wallet metadata.
  - src/analytics/index.ts sets `disabled` when IS_TEST is true or the 'doNotTrack' device setting is on. Nothing is sent in either case. Events are queued until the PostHog client is ready.
  - The scanner found no `reset` call. src/analytics/index.ts has no reset method, and the identify id is a persistent deviceId, so identity is device-scoped.
  - Every call reaches the scanner as analytics.track(analytics.event.<key>, props). The name is a key of the `event` map in src/analytics/event.ts, and the sent name is that key's string value (for example positionsError is 'positions.error'). The property types are in EventProperties in the same file.
  - src/analytics/index.ts (PostHog via posthog-react-native) adds walletAddressHash and walletType to every track and identify call. On Android it also adds device_brand, device_manufacturer and device_model. track also adds appsflyer_id when AppsFlyer has a uid. It identifies with the device id, not a user or wallet id.
  - src/analytics/index.ts also sends the PostHog events 'Application Updated' (captureVersionUpdate) and PostHog's automatic 'Application Installed', which is dropped for returning users. The wrapper also has screen(route, params), which sends PostHog screen events.
  - analytics.disable() and the doNotTrack setting stop all sending.
  - Every call site passes the event name as `analytics.event.<key>`. The scanner can't follow that lookup, so it reports 0 events. `analytics` is an instance of the Analytics class in src/analytics/index.ts, and its `event` field is the `event` constant map in src/analytics/event.ts:38-364. The map is `as const`, with literal string values and no computed names. The lines above are only the ones the scanner listed. Any other `analytics.track(analytics.event.X)` call can be resolved by reading that map. EventProperties in the same file (line 412) types each event's properties.
  - src/analytics/index.ts is a PostHog (posthog-react-native) wrapper. Its track() adds walletAddressHash and walletType on every event. On Android it also adds device_brand, device_manufacturer and device_model. It adds appsflyer_id when an AppsFlyer uid exists.
  - src/analytics/index.ts also sends events on its own: captureVersionUpdate sends 'Application Updated' with previous_version and previous_build. The PostHog SDK sends lifecycle events such as 'Application Installed', and the before_send hook drops it for returning users. analytics.screen(route, params) sends PostHog screen events named by the route.
  - analytics.identify uses the deviceId as the PostHog distinct id and attaches the hashed wallet address. It does not use a wallet address or email as the id. analytics.enable and analytics.disable opt the PostHog client in or out. Tracking is off in tests and when device storage has doNotTrack.
  - Every call site passes `analytics.event.<key>`, a member of the `event` constants object in src/analytics/event.ts. The sent name is that object's string value, not the key. src/analytics/event.ts also types each event's properties in `EventProperties`, which is a ready source for the registry's property lists.
  - src/analytics/index.ts (PostHog via posthog-react-native) adds walletAddressHash and walletType to every track call. On Android it also adds device_brand, device_manufacturer and device_model. When an AppsFlyer uid exists it adds appsflyer_id.
  - src/analytics/index.ts sends 'Application Updated' (previous_version, previous_build) by itself from captureVersionUpdate(). PostHog's own autocapture of 'Application Installed' is dropped in before_send for returning users.
  - analytics.screen(route, params) sends PostHog screen events. The call src/navigation/onNavigationStateChange.js:130 passes a route name chosen at runtime, so it is not a literal name. No tracker is registered for it.
  - The wrapper identifies with the device id (analytics.init({ deviceId })), not a user or wallet id. The wallet address hash is sent only as a trait.
  - The analytics.track and analytics.identify entries registered with `from: @/analytics` never matched anything: the scanner counted 0 events. The new entries without `from` replace them. src/analytics/index.ts is the only file that calls PostHog.
  - Every tracking call in the app uses the form analytics.track(analytics.event.<key>, props). The name is the value of a key in the `event` map in src/analytics/event.ts, and the property types are in EventProperties in the same file. The scanner doesn't read this member reference as a literal. In round 2 only 4 files were listed, but about 145 files use this form. If the whole set shows up as unreadable, resolve it through src/analytics/event.ts.
  - src/analytics/index.ts sends through PostHog (posthog-react-native). track() adds walletAddressHash, walletType and appsflyer_id (when AppsFlyer has a uid) to every event. On Android it also adds device_brand, device_manufacturer and device_model.
  - src/analytics/index.ts:169 sends 'Application Updated' on its own, with previous_version and previous_build, when the version changes but the build number doesn't. PostHog itself sends 'Application Installed' (suppressed for returning users in before_send) and the other lifecycle events.
  - analytics.screen(route, params) in src/analytics/index.ts sends PostHog screen events. It's called from src/navigation/onNavigationStateChange.js:130 with the route name at runtime.
  - Tracking is off when IS_TEST is set or the user has set doNotTrack, and analytics.disable() drops queued events. Events sent before PostHog is ready are queued and flushed when it starts.
  - The identify wrapper uses the deviceId as the PostHog distinct id, with traits merged with walletAddressHash and walletType. The wrapper has no reset or alias method.
  - Every analytics.track call in src/ passes a key of the `event` constant in src/analytics/event.ts (analytics.event.X or event.X). None passes a string literal, so the registry's event names are the string values in that file, not the key names.
  - The Analytics class in src/analytics/index.ts wraps posthog-react-native. It sends analytics.screen(route, params) as PostHog screen events. It also sends an 'Application Updated' capture from captureVersionUpdate (line 169) when the version changes but the build number does not.
  - track() also adds appsflyer_id when AppsFlyer has a uid. On Android, track(), screen() and identify() add device_brand, device_manufacturer and device_model.
  - The wrapper drops the PostHog 'Application Installed' event for returning users (before_send, line 200).
  - Identity uses the deviceId from analytics.init as the PostHog distinct id, not a wallet address. analytics.identify only sends traits, and nothing in the code calls reset or alias.
  - The tracker entries `analytics.track` and `analytics.identify` with from '@/analytics' still reported 0 events and flagged calls as vendor SDK calls. The code imports `{ analytics }` as a named import of a class instance, which may be why. The 297 `analytics.track(...event.X...)` calls in 138 files may all be unseen by the scanner. Check whether the next scan sees them.
  - Event names are not literals. Callers write `analytics.event.<key>` or `event.<key>`, and src/analytics/event.ts maps each key to its string. The `explained` entries list the resolved strings. EventProperties in the same file types each event's properties.
  - src/analytics/index.ts (the wrapper) sends a few things on its own. `Application Updated` with previous_version and previous_build is captured in captureVersionUpdate, when the app version changed but the build number didn't. PostHog's automatic `Application Installed` event is dropped for returning users in before_send.
  - analytics.track adds appsflyer_id to every event when AppsFlyer has a uid. On Android it also adds device_brand, device_manufacturer and device_model to every event and identify (getDefaultMetadata).
  - analytics.screen(route, params) in src/analytics/index.ts sends PostHog screen events. It isn't registered as a tracker here.
  - analytics.identify(traits) identifies with the device id (not a wallet or email), and the traits are the profile properties set. The wrapper does nothing when the user has doNotTrack set or in tests (IS_TEST).
  - The wrapper is src/analytics/index.ts. It wraps posthog-react-native and also starts AppsFlyer (src/analytics/appsflyer.ts). All calls elsewhere go through `analytics` from '@/analytics'.
  - Callers pass analytics.track a key of the `event` map in src/analytics/event.ts (analytics.event.<key>), not a string literal. The map values are the sent names, such as 'add_favorite_token', 'add_wallet_flow.started' and 'Tapped Watch Address'. Names are in two styles: snake/dot style and Title Case. Each EventProperties entry in that file defines the properties for its event.
  - analytics.track adds walletAddressHash and walletType from the wrapper's wallet context, and appsflyer_id when AppsFlyer has a uid. On Android it also adds device_brand, device_manufacturer and device_model (src/analytics/index.ts:102-108, 145-158).
  - The wrapper sends on its own: 'Application Updated' with previous_version and previous_build (index.ts:160-175, from captureVersionUpdate), and PostHog's automatic 'Application Installed', which is dropped for returning users (index.ts:200). It also has analytics.screen(route, params), which sends PostHog screen events named by the route. Those are not product events written in callers.
  - The wrapper identifies with the random device id (deviceId), not a wallet or account id. The wallet address hash goes in as a property (index.ts:77-86). The wrapper has no reset or alias.
  - Tracking is off when IS_TEST is set or the user turned on doNotTrack (index.ts:48). Events queue until PostHog is initialised. When POSTHOG_API_KEY or POSTHOG_HOST is missing, queued calls are dropped.
  - src/redux/settings.ts:121 and src/App.tsx send a temporary debug.identify_probe event (event.debugIdentifyProbe, 'debug.identify_probe'), marked in a code comment for removal after FEPLAT-67.
  - Every call site (about 305 across 145 files) names its event as analytics.event.<key>, a lookup into the map in src/analytics/event.ts. The scanner cannot read it as a literal name. Only the flagged lines are explained here, and the other call sites need the same treatment.
  - The 'vendor SDK called outside a wrapper' flags on these lines look like the same issue. They are calls to the wrapper object (analytics from '@/analytics', src/analytics/index.ts), not to posthog-react-native. Nothing in these files imports the vendor SDK.
  - src/analytics/index.ts (the wrapper, PostHog via posthog-react-native) sends some things on its own. It sends analytics.screen(route, params), which is a PostHog screen event. It captures 'Application Updated' with previous_version and previous_build after a version change. It adds appsflyer_id to every track call when AppsFlyer has a uid. On Android it adds device_brand, device_manufacturer and device_model to every track, screen and identify call. It bootstraps PostHog with the device id as distinctId, and drops 'Application Installed' for returning users.
  - analytics.identify(userProperties) in src/analytics/index.ts identifies with the device id, not a user id, and sends the traits as profile properties.
  - src/analytics/index.ts is the one wrapper. It is the Analytics class over posthog-react-native, exported as `analytics`. Call sites name events either as `analytics.event.<key>` or through a named import of `event`. Both are keys of the map in src/analytics/event.ts, whose values are the real PostHog names (e.g. 'claim_claimable.failed'). Calls that use `analytics.event.X` can't be read by name, because the name sits behind a property of the wrapper object.
  - analytics.track in src/analytics/index.ts adds walletAddressHash and walletType on every event, plus device_brand, device_manufacturer and device_model on Android. It adds appsflyer_id when AppsFlyer has a uid. A walletContext argument can add more.
  - src/analytics/index.ts sends 'Application Updated' straight from the PostHog client (captureVersionUpdate, with previous_version and previous_build). PostHog itself sends 'Application Installed', which before_send drops for returning users.
  - analytics.screen(route, params) in the wrapper sends a PostHog screen event. It is called from src/navigation/onNavigationStateChange.js and src/state/wallets/initializeWallet.ts. The scanner has no call entry for it.
  - src/performance/tracking/index.ts:86 and src/__swaps__/utils/trackSwapEvent.ts:71 pass a variable as the event name to analytics.track. Those are forwarding wrappers: their callers choose the name.
  - Analytics is disabled when IS_TEST is set or the user has turned on doNotTrack, so no calls are sent in those cases.
  - Every analytics.track call names its event with analytics.event.<key>. `event` is a const map of keys to strings in src/analytics/event.ts, and EventProperties in the same file types each event's properties. The scanner doesn't follow this member lookup, so the listed refs are credited by `explained`. The same pattern is probably used by the other ~145 files that call analytics.track.
  - src/analytics/index.ts (PostHog via posthog-react-native, the wrapper) adds walletAddressHash and walletType from setWalletContext, device_brand/device_manufacturer/device_model on Android, and appsflyer_id when AppsFlyer has a uid, to every tracked event.
  - src/analytics/index.ts also sends events on its own: analytics.screen(route, params) sends a PostHog screen event, and captureVersionUpdate sends 'Application Updated' (previous_version, previous_build) on start-up. PostHog itself sends 'Application Installed' and similar lifecycle events, and before_send drops 'Application Installed' for returning users.
  - Analytics is disabled when IS_TEST or the doNotTrack device setting is on, and the PostHog client isn't created without POSTHOG_API_KEY and POSTHOG_HOST.
  - Every call site reads analytics.track(analytics.event.<key>, props) (about 305 calls in 145 files). The `event` object in src/analytics/event.ts maps each key to a literal string, and EventProperties in the same file types each event's properties. The names are literals, but the scanner cannot read them through the member access. A registry builder can read the full name list and property types from event.ts directly.
  - src/analytics/index.ts (Analytics.track) sends through PostHog (posthog-react-native) and adds walletAddressHash and walletType, plus device_brand, device_manufacturer and device_model on Android. It adds appsflyer_id when an AppsFlyer uid exists.
  - src/analytics/index.ts also sends 'Application Updated' (previous_version, previous_build) on its own from captureVersionUpdate. Its before_send hook drops 'Application Installed' for returning users. PostHog's own autocapture of 'Application Installed' is therefore filtered.
  - Analytics.screen(route, params) in src/analytics/index.ts sends PostHog screen events. Analytics.identify uses the deviceId as the distinct id and attaches the hashed wallet address. The wrapper has no reset or alias.
  - Analytics.enable() and disable() opt PostHog in and out, and the choice persists in device storage as doNotTrack. Events are queued until the client is ready, and dropped while analytics is disabled.
  - Every analytics.track call passes a key of the const `event` map in src/analytics/event.ts (as `analytics.event.X` or `event.X`), never a string literal: about 296 `analytics.event.` and 20 `analytics.track(event.` call sites. The actual event names are the string values in that file, and the registry should be drafted from event.ts, since the scanner can't read them from the calls.
  - src/analytics/index.ts wraps posthog-react-native: track -> client.capture, identify -> client.identify(deviceId, traits) (the id is a device id, not a wallet address), screen -> client.screen (a PostHog $screen event). analytics.screen is called from src/navigation/onNavigationStateChange.js:130 with a runtime route name, so screen views have no event name in the code and are not in `calls`.
  - The wrapper adds walletAddressHash and walletType to every track and screen call, device_brand, device_manufacturer and device_model on Android, and appsflyer_id when AppsFlyer has a uid.
  - The wrapper sends 'Application Updated' by itself (index.ts:169, with previous_version and previous_build) when the version changes but the build number doesn't. The PostHog client's lifecycle capture of 'Application Installed' is dropped for returning users (before_send, index.ts:200).
  - Analytics is switched off in tests (IS_TEST) and when the device setting doNotTrack is on; calls queue until the PostHog client is ready.
  - No reset or alias call exists in the code: the scan found none and the wrapper has no reset method.
  - src/navigation/onNavigationStateChange.js:130 sends a screen event (analytics.screen(routeName, params)) on every route change, so screen views arrive as one event per Routes value.
  - The identify calls the scanner still flags (Chart.tsx, useIdentifyAppIcon.ts, notifications/analytics.ts, redux/settings.ts:115) all use the '@/analytics' wrapper. They are already counted as user properties and are listed as exceptions.
  - src/analytics/index.ts track() also adds appsflyer_id to every event when AppsFlyer has a uid, and on Android adds device_brand, device_manufacturer and device_model to track, screen and identify calls (getDefaultMetadata). screen() and track() accept a walletContext argument whose walletAddressHash and walletType override the instance's.
  - src/analytics/index.ts sends 'Application Updated' by itself (captureVersionUpdate, line 169) with previous_version and previous_build when the version changes but the build number does not. The PostHog React Native client also sends its own lifecycle events (e.g. 'Application Installed'); before_send (line 197-202) drops 'Application Installed' for returning users and drops everything when analytics is disabled.
  - The wrapper is a no-op when IS_TEST or the doNotTrack device setting is on, and queues calls until the PostHog client is ready (src/analytics/index.ts:47-52, 219-228).
  - analytics.screen is called only from src/navigation/onNavigationStateChange.js:130, so every screen view goes through one place with the route name as the event name.

## Surfaces and what people do on them

Code found 130 surfaces from the frameworks' own conventions: 130 screens. 348 actions people take on them, 118 core; **89 core or secondary actions have no event**. 9 surfaces have no action named (nothing to act on, or not checked). All of it is in `_map/`, one file per area, which the PR check reads to see what a change can reach.

Each event's call was followed back through the code to what triggers it, and checked: 84 fire after success, 68 fire before the request, 46 fire after failure, 14 fire on view, 12 fire on mount. 17 events have a link code couldn't follow (`certainty: unsure`), so a PR touching them gets a closer read. Gaps with a proposed event are in the migration plan.

| Area | Surface | Action | Importance | Events |
|---|---|---|---|---|
| action | `ActionViewController` | Open a shared link or text in Rainbow's dapp browser | secondary | **none** |
| asset_list | `Routes.WALLET_SCREEN` | View wallet token list | core | `token_list` |
| asset_list | `Routes.SELECT_UNIQUE_TOKEN_SHEET` | Select an NFT from the list | secondary | **none** |
| asset_list | `Routes.WALLET_SCREEN` | Open My QR Code (receive) | secondary | `Tapped "My QR Code"` |
| cards | `Routes.AIRDROPS_SHEET` | Open an airdrop to claim | core | **none** |
| cards | `Routes.PIN_AUTHENTICATION_SCREEN` | Create and confirm a PIN | core | **none** |
| cards | `Routes.PIN_AUTHENTICATION_SCREEN` | Enter PIN to authenticate | core | **none** |
| cards | `Routes.RNBW_MEMBERSHIP_TIERS_SHEET` | Browse membership tiers | secondary | **none** |
| discover | `Routes.SWIPE_LAYOUT` | Claim RNBW airdrop | core | `rnbw_airdrop.claim`, `rnbw_airdrop.claim.failed` |
| discover | `Routes.SWIPE_LAYOUT` | Claim RNBW rewards | core | `rnbw_rewards.claim`, `rnbw_rewards.claim.failed` |
| discover | `Routes.SWIPE_LAYOUT` | Initialize wallet on launch | core | `wallet_initialization.failed` |
| discover | `Routes.DISCOVER_SCREEN` | Place a prediction market order | core | `discover.prediction_order_pressed` |
| discover | `Routes.DISCOVER_SCREEN` | Sign in to Cash | core | `cash.sign_in_submitted`, `cash.sign_in_succeeded`, `cash.sign_in_failed`, `cash.sign_in_cancelled` |
| discover | `Routes.DISCOVER_SCREEN` | Create a token with the token launcher | core | `token_launcher.step_changed`, `token_launcher.token_created`, `token_launcher.creation_failed` |
| discover | `Routes.SWIPE_LAYOUT` | Switch between app tabs | secondary | **none** |
| discover | `Routes.SWIPE_LAYOUT` | View wallet token list | secondary | `token_list` |
| discover | `Routes.SWIPE_LAYOUT` | Open My QR Code | secondary | `Tapped "My QR Code"` |
| discover | `Routes.SWIPE_LAYOUT` | Open Discover search | secondary | `Tapped Search` |
| discover | `Routes.SWIPE_LAYOUT` | Open a favorite dapp from the browser homepage | secondary | `browser.tap_favorite` |
| discover | `Routes.SWIPE_LAYOUT` | Open a trending dapp | secondary | `browser.trending_dapp_pressed` |
| discover | `Routes.SWIPE_LAYOUT` | Resolve a pending transaction (watcher) | secondary | `pending_transaction.resolved` |
| discover | `Routes.SWIPE_LAYOUT` | Resolve assets for a mined transaction (watcher) | secondary | `mined_transaction.assets_resolved`, `mined_transaction.assets_timed_out` |
| discover | `Routes.DISCOVER_SCREEN` | Open Discover search | secondary | `Tapped Search` |
| discover | `Routes.DISCOVER_SCREEN` | Search for tokens | secondary | `Search Query`, `token_list` |
| discover | `Routes.DISCOVER_SCREEN` | Open a Discover card | secondary | `discover.card_pressed`, `surface.interaction` |
| discover | `Routes.DISCOVER_SCREEN` | Add a favorite dapp | secondary | `browser.add_favorite` |
| discover | `Routes.DISCOVER_SCREEN` | Add a favorite token | secondary | `add_favorite_token` |
| expanded_state | `Routes.MODAL_SCREEN` | Name and create a new wallet group | core | `add_new_wallet_group.name`, `wallet_create.succeeded`, `wallet_create.failed` |
| expanded_state | `Routes.MODAL_SCREEN` | Submit wallet profile (name and color) | secondary | `Tapped "Submit" on Wallet Profile modal` |
| expanded_state | `Routes.MODAL_SCREEN` | Save or edit a contact profile | secondary | **none** |
| expanded_state | `Routes.EXPANDED_ASSET_SHEET` | View NFT token details | secondary | `token_details.nft` |
| expanded_state | `Routes.EXPANDED_ASSET_SHEET` | Hide or unhide an NFT | secondary | `Toggled an NFT as Hidden` |
| expanded_state | `Routes.EXPANDED_ASSET_SHEET` | Tap buy from the asset sheet | secondary | `buy_button.pressed` |
| expanded_state | `Routes.CUSTOM_GAS_SHEET` | Choose a gas price option | secondary | `Updated Gas Price` |
| expanded_state | `Routes.CUSTOM_GAS_SHEET` | Enter custom gas fees | secondary | **none** |
| features | `Routes.SEND_SHEET` | Enter a recipient and select an asset to send | core | **none** |
| features | `Routes.SEND_SHEET` | Submit a send transaction | core | `Sent transaction` |
| features | `Routes.RESTORE_SHEET` | Choose a backup to restore a wallet | core | **none** |
| features | `Routes.RESTORE_SHEET` | Restore a wallet from a cloud backup | core | `wallet_create.succeeded`, `wallet_create.failed`, `backup.error` |
| features | `Routes.NOTIFICATION_PERMISSION_SCREEN` | Enable push notifications | core | `notification_permission.requested`, `notification_permission.granted`, `notification_permission.denied` |
| features | `Routes.PAIR_HARDWARE_WALLET_NAVIGATOR` | Start pairing a hardware wallet | core | `pair_hw_wallet_nav.entered` |
| features | `Routes.PAIR_HARDWARE_WALLET_NAVIGATOR` | Search for and select a Ledger device | core | **none** |
| features | `Routes.REGISTER_ENS_NAVIGATOR` | Register an ENS name | core | **none** |
| features | `Routes.BACKUP_SHEET` | Create a cloud backup | core | `Choose Password Step`, `Tapped "Confirm Backup"`, `Backup Complete`, `backup.error`, `Saved backup password on iCloud`, `Didn't save backup password on iCloud`, `Error during Google Drive Backup`, `Error during iCloud Backup`, `Error updating Backup status`, `iCloud not enabled`, `View how to Enable iCloud`, `Ignore how to enable iCloud` |
| features | `Routes.BACKUP_SHEET` | Restore wallet from cloud backup | core | **none** |
| features | `Routes.CONFIRM_REQUEST` | Approve a dapp request | core | `request.approved` |
| features | `Routes.CONFIRM_REQUEST` | Reject a dapp request | core | `request.rejected` |
| features | `Routes.POLYMARKET_DEPOSIT_SCREEN` | Deposit funds to Polymarket | core | `predictions.deposit`, `predictions.deposit.failed` |
| features | `Routes.PERPS_WITHDRAWAL_SCREEN` | Withdraw funds from perps | core | `perps.withdrew`, `perps.withdraw.failed` |
| features | `Routes.POLYMARKET_WITHDRAWAL_SCREEN` | Withdraw funds from Polymarket | core | `predictions.withdrawal`, `predictions.withdrawal.failed` |
| features | `Routes.SEND_SHEET_NAVIGATOR` | Send a token or NFT to a recipient | core | `Sent transaction` |
| features | `Routes.PERPS_NAVIGATOR` | Open a perps position | core | `perps.opened_position`, `perps.open_position.failed`, `perps.open_position.canceled` |
| features | `Routes.CREATE_TRIGGER_ORDER_BOTTOM_SHEET` | Create a take-profit or stop-loss trigger order | core | `perps.trigger_order_created`, `perps.trigger_order.failed` |
| features | `Routes.CLOSE_POSITION_BOTTOM_SHEET` | Close a perps position | core | `perps.closed_position`, `perps.close_position.failed` |
| features | `Routes.PERPS_ADD_TO_POSITION_SHEET` | Add to a perps position | core | `perps.added_to_position`, `perps.added_to_position.failed` |
| features | `Routes.CASH_DEPOSIT_INTRO_PANEL` | Set up cash account | core | **none** |
| features | `Routes.CASH_DEPOSIT_INTRO_PANEL` | Sign in to existing cash account | core | **none** |
| features | `Routes.CASH_DEPOSIT_SETUP_SCREEN` | Submit phone number for Cash account | core | `cash.phone_submitted`, `cash.phone_submit_failed`, `cash.phone_already_registered` |
| features | `Routes.CASH_DEPOSIT_SETUP_SCREEN` | Verify phone with code | core | `cash.phone_verified`, `cash.phone_verify_failed` |
| features | `Routes.CASH_DEPOSIT_SETUP_SCREEN` | Submit identity verification (KYC) | core | `cash.kyc_submitted`, `cash.kyc_approved`, `cash.kyc_awaiting_decision`, `cash.kyc_failed` |

And 204 more in `_map/`.

## Status of each item

Worked out from the scan of the code this audit read, not stored in the registry: the registry says only whether an item is planned or deprecated. 264 implemented.

| Item | Type | Status | Call in the code |
|---|---|---|---|
| `add_favorite_token` | event | implemented | `src/resources/favorites.ts:178` |
| `add_new_wallet_group.name` | event | implemented | `src/components/expanded-state/NewWalletGroupState.tsx:20` |
| `add_wallet_flow.started` | event | implemented | `src/screens/AddWalletSheet.tsx:45`, `src/screens/AddWalletSheet.tsx:49`, `src/screens/AddWalletSheet.tsx:58` and 6 more |
| `analytics_tracking.disabled` | event | implemented | `src/screens/SettingsSheet/components/PrivacySection.tsx:35`, `src/screens/SettingsSheet/components/PrivacySection.tsx:42` |
| `analytics_tracking.enabled` | event | implemented | `src/screens/SettingsSheet/components/PrivacySection.tsx:35`, `src/screens/SettingsSheet/components/PrivacySection.tsx:42` |
| `app_icon_unlock_sheet.cta_pressed` | event | implemented | `src/features/app-icon/screens/AppIconUnlockSheet.tsx:39`, `src/features/app-icon/screens/AppIconUnlockSheet.tsx:43` |
| `app_icon_unlock_sheet.viewed` | event | implemented | `src/features/app-icon/screens/AppIconUnlockSheet.tsx:39`, `src/features/app-icon/screens/AppIconUnlockSheet.tsx:43` |
| `app_store_review.prompted` | event | implemented | `src/utils/reviewAlert.ts:108` |
| `appIcon` | user property | implemented | `src/hooks/useIdentifyAppIcon.ts:10` |
| `Approved new WalletConnect session` | event | implemented | `src/features/wallet-connect/handlers/onSessionProposal.ts:38`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:158` and 1 more |
| `Backup Complete` | event | implemented | `src/features/backup/hooks/useCreateBackup.ts:65`, `src/features/backup/hooks/useCreateBackup.ts:92` |
| `backup.error` | event | implemented | `src/features/backup/backup.ts:187`, `src/features/backup/backup.ts:532`, `src/features/backup/backup.ts:535` |
| `BackupSheet shown` | event | implemented | `src/features/backup/components/BackupSheetSection.tsx:56` |
| `branchCampaign` | user property | implemented | `src/utils/branch.ts:127` |
| `branchReferrer` | user property | implemented | `src/utils/branch.ts:127` |
| `branchReferringLink` | user property | implemented | `src/utils/branch.ts:127` |
| `browser.add_favorite` | event | implemented | `src/features/dapp-browser/stores/favoriteDappsStore.ts:50` |
| `browser.tap_favorite` | event | implemented | `src/features/dapp-browser/components/Homepage.tsx:226`, `src/features/dapp-browser/components/Homepage.tsx:334` |
| `browser.trending_dapp_pressed` | event | implemented | `src/features/dapp-browser/components/Homepage.tsx:226`, `src/features/dapp-browser/components/Homepage.tsx:334` |
| `buy_button.pressed` | event | implemented | `src/components/cards/EthCard.tsx:74`, `src/components/cards/EthCard.tsx:90`, `src/components/sheet/sheet-action-buttons/BuyActionButton.tsx:32` |
| `candleResolution` | user property | implemented | `src/components/value-chart/Chart.tsx:282` |
| `card.pressed` | event | implemented | `src/components/cards/EthCard.tsx:74`, `src/components/cards/EthCard.tsx:90`, `src/components/cards/LearnCard.tsx:31` and 2 more |
| `cash.amount_entered` | event | implemented | `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477`, `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:495` |
| `cash.buy_completed` | event | implemented | `src/features/cash/stores/cashBuyOrderStore.ts:123`, `src/features/cash/stores/cashBuyOrderStore.ts:146`, `src/features/cash/stores/cashBuyOrderStore.ts:233` and 1 more |
| `cash.buy_failed` | event | implemented | `src/features/cash/stores/cashBuyOrderStore.ts:123`, `src/features/cash/stores/cashBuyOrderStore.ts:146`, `src/features/cash/stores/cashBuyOrderStore.ts:233` and 1 more |
| `cash.buy_submitted` | event | implemented | `src/features/cash/stores/cashBuyOrderStore.ts:123`, `src/features/cash/stores/cashBuyOrderStore.ts:146`, `src/features/cash/stores/cashBuyOrderStore.ts:233` and 1 more |
| `cash.card_link_failed` | event | implemented | `src/features/cash/stores/cardLinkFlowStore.ts:47`, `src/features/cash/stores/cardLinkFlowStore.ts:58` |
| `cash.card_linked` | event | implemented | `src/features/cash/stores/cardLinkFlowStore.ts:47`, `src/features/cash/stores/cardLinkFlowStore.ts:58` |
| `cash.deposit_intro_viewed` | event | implemented | `src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40` |
| `cash.deposit_signup_paused_viewed` | event | implemented | `src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40` |
| `cash.existing_account_recovery_selected` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126` and 3 more |
| `cash.kyc_approved` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 10 more |
| `cash.kyc_awaiting_decision` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 10 more |
| `cash.kyc_failed` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 10 more |
| `cash.kyc_submitted` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 6 more |
| `cash.lost_passkey_recovery_selected` | event | implemented | `src/features/cash/screens/add-cash-sheet/ReauthenticateContent.tsx:51` |
| `cash.passkey_added` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73` |
| `cash.passkey_failed` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73` |
| `cash.passkey_submitted` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73` |
| `cash.phone_already_registered` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126` and 3 more |
| `cash.phone_resend_failed` | event | implemented | `src/features/cash/stores/verifyPhoneFlowStore.ts:106`, `src/features/cash/stores/verifyPhoneFlowStore.ts:127`, `src/features/cash/stores/verifyPhoneFlowStore.ts:155` and 1 more |
| `cash.phone_submit_failed` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126` and 3 more |
| `cash.phone_submitted` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126` and 16 more |
| `cash.phone_verified` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 10 more |
| `cash.phone_verify_failed` | event | implemented | `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124` and 10 more |
| `cash.sign_in_cancelled` | event | implemented | `src/features/cash/services/cashSignInService.ts:53`, `src/features/cash/services/cashSignInService.ts:68`, `src/features/cash/services/cashSignInService.ts:76` and 1 more |
| `cash.sign_in_failed` | event | implemented | `src/features/cash/services/cashSignInService.ts:53`, `src/features/cash/services/cashSignInService.ts:68`, `src/features/cash/services/cashSignInService.ts:76` and 1 more |
| `cash.sign_in_submitted` | event | implemented | `src/features/cash/services/cashSignInService.ts:53`, `src/features/cash/services/cashSignInService.ts:68`, `src/features/cash/services/cashSignInService.ts:76` and 1 more |
| `cash.sign_in_succeeded` | event | implemented | `src/features/cash/services/cashSignInService.ts:53`, `src/features/cash/services/cashSignInService.ts:68`, `src/features/cash/services/cashSignInService.ts:76` and 1 more |
| `cash.wallet_check_failed` | event | implemented | `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477`, `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:495` |
| `cash.wallet_link_failed` | event | implemented | `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36`, `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:47` |
| `cash.wallet_link_prompted` | event | implemented | `src/features/cash/screens/add-wallet-sheet/AddWalletSheet.tsx:87` |
| `cash.wallet_linked` | event | implemented | `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36`, `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:47` |
| `Changed Candle Resolution` | event | implemented | `src/components/value-chart/Chart.tsx:272`, `src/components/value-chart/Chart.tsx:283` |
| `Changed Chart Type` | event | implemented | `src/components/value-chart/Chart.tsx:272`, `src/components/value-chart/Chart.tsx:283` |
| `Changed Global Notification Settings` | event | implemented | `src/notifications/analytics.ts:11`, `src/notifications/analytics.ts:17` |
| `Changed language` | event | implemented | `src/screens/SettingsSheet/components/LanguageSection.tsx:24` |
| `Changed native currency` | event | implemented | `src/screens/SettingsSheet/components/CurrencySection.tsx:40` |
| `Changed native currency input in Send flow` | event | implemented | `src/features/transfer/screens/SendSheet.tsx:357`, `src/features/transfer/screens/SendSheet.tsx:378`, `src/features/transfer/screens/SendSheet.tsx:554` and 1 more |
| `Changed network` | event | implemented | `src/screens/SettingsSheet/components/NetworkSection.tsx:30` |
| `Changed token input in Send flow` | event | implemented | `src/features/transfer/screens/SendSheet.tsx:357`, `src/features/transfer/screens/SendSheet.tsx:378`, `src/features/transfer/screens/SendSheet.tsx:554` and 1 more |
| `chartType` | user property | implemented | `src/components/value-chart/Chart.tsx:271` |
| `Choose Password Step` | event | implemented | `src/features/backup/components/BackupCloudStep.tsx:58` |
| `claim_claimable.failed` | event | implemented | `src/screens/Airdrops/utils.ts:121`, `src/screens/Airdrops/utils.ts:150`, `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:74` and 11 more |
| `claim_claimable.succeeded` | event | implemented | `src/screens/Airdrops/utils.ts:121`, `src/screens/Airdrops/utils.ts:150`, `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:74` and 11 more |
| `claimable_panel.opened` | event | implemented | `src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88` |
| `claimablesAmount` | user property | implemented | `src/resources/addys/claimables/query.ts:83` |
| `claimablesUSDValue` | user property | implemented | `src/resources/addys/claimables/query.ts:83` |
| `Clicked "Max" in Send flow input` | event | implemented | `src/features/transfer/components/SendAssetFormField.js:71` |
| `Completed ENS registration` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `currency` | user property | implemented | `src/redux/settings.ts:115`, `src/redux/settings.ts:238` |
| `debug.identify_probe` | event | implemented | `src/App.tsx:177`, `src/App.tsx:198`, `src/redux/settings.ts:121` |
| `Didn't save backup password on iCloud` | event | implemented | `src/features/backup/backup.ts:187`, `src/features/backup/backup.ts:532`, `src/features/backup/backup.ts:535` |
| `discover.card_pressed` | event | implemented | `src/features/discover/utils/trackDiscoverCardPress.ts:28` |
| `discover.carousel_scrolled` | event | implemented | `src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17` |
| `discover.prediction_order_pressed` | event | implemented | `src/features/discover/components/markets/layouts/MarketCarousel.tsx:87`, `src/features/discover/components/markets/layouts/MarketGrid.tsx:83`, `src/features/discover/components/markets/layouts/MarketList.tsx:82` |
| `discover.section_pressed` | event | implemented | `src/features/discover/components/PredictionEventsSection.tsx:87`, `src/features/discover/components/SectionLayout.tsx:41` |
| `discover.tab_pressed` | event | implemented | `src/features/discover/components/DiscoverHeader.tsx:134` |
| `Edited ENS records` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `enabledTestnets` | user property | implemented | `src/redux/settings.ts:115` |
| `Error during Google Drive Backup` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `Error during iCloud Backup` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `Error updating Backup status` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `error_boundary.reset` | event | implemented | `src/app/error-boundary/ErrorBoundary.tsx:14`, `src/app/error-boundary/ErrorBoundary.tsx:21` |
| `error_boundary.viewed` | event | implemented | `src/app/error-boundary/ErrorBoundary.tsx:14`, `src/app/error-boundary/ErrorBoundary.tsx:21` |
| `Extended ENS` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `f2c.provider.flow_completed` | event | implemented | `src/handlers/deeplinks.ts:172`, `src/handlers/deeplinks.ts:178` |
| `f2c.provider.flow_opened` | event | implemented | `src/screens/FiatOnRamp/ProviderListItem.tsx:35` |
| `First App Open` | event | implemented | `src/App.tsx:177`, `src/App.tsx:198` |
| `hardwareAccounts` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `hasImported` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `iCloud not enabled` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `Ignore how to enable iCloud` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `Imported seed phrase` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `importedPrivateKeys` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `importedRecoveryPhrases` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `Initiated ENS registration` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `installSource` | user property | implemented | `src/App.tsx:173` |
| `insufficient_native_asset_for_action` | event | implemented | `src/__swaps__/screens/Swap/providers/SyncSwapStateAndSharedValues.tsx:111`, `src/features/gas/hooks/useGas.ts:92` |
| `language` | user property | implemented | `src/redux/settings.ts:153`, `src/redux/settings.ts:223` |
| `learn_article.opened` | event | implemented | `src/screens/LearnWebViewScreen.tsx:35`, `src/screens/LearnWebViewScreen.tsx:51` |
| `learn_article.shared` | event | implemented | `src/screens/LearnWebViewScreen.tsx:35`, `src/screens/LearnWebViewScreen.tsx:51` |
| `ledgerDevices` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `Manually disconnected from WalletConnect connection` | event | implemented | `src/features/wallet-connect/components/WalletConnectV2ListItem.tsx:112` |
| `mined_transaction.assets_resolved` | event | implemented | `src/hooks/useWatchAssetUpdates.ts:70`, `src/hooks/useWatchAssetUpdates.ts:173` |
| `mined_transaction.assets_timed_out` | event | implemented | `src/hooks/useWatchAssetUpdates.ts:70`, `src/hooks/useWatchAssetUpdates.ts:173` |
| `mostUsedNetworks` | user property | implemented | `src/features/network/stores/networkSwitcherStore.ts:49` |
| `network_status.offline` | event | implemented | `src/hooks/useIsOffline.ts:39`, `src/hooks/useIsOffline.ts:47`, `src/hooks/useIsOffline.ts:52` |
| `network_status.reconnected` | event | implemented | `src/hooks/useIsOffline.ts:39`, `src/hooks/useIsOffline.ts:47`, `src/hooks/useIsOffline.ts:52` |
| `New WalletConnect pairing` | event | implemented | `src/features/wallet-connect/handlers/onSessionProposal.ts:38`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:158` and 1 more |
| `notification_permission.denied` | event | implemented | `src/features/notifications/screens/NotificationPermissionScreen.tsx:57`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:65`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:68` and 1 more |
| `notification_permission.granted` | event | implemented | `src/features/notifications/screens/NotificationPermissionScreen.tsx:57`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:65`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:68` and 1 more |
| `notification_permission.requested` | event | implemented | `src/features/notifications/screens/NotificationPermissionScreen.tsx:57`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:65`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:68` and 1 more |
| `notification_permission.skipped` | event | implemented | `src/features/notifications/screens/NotificationPermissionScreen.tsx:57`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:65`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:68` and 1 more |
| `notificationsPermissionStatus` | user property | implemented | `src/notifications/analytics.ts:24` |
| `numberOfImportedWalletsWithNotificationsTurnedOn` | user property | implemented | `src/notifications/analytics.ts:118` |
| `numberOfWatchedWalletsWithNotificationsTurnedOn` | user property | implemented | `src/notifications/analytics.ts:118` |
| `Opened asset from position` | event | implemented | `src/features/positions/screens/PositionSheet.tsx:68`, `src/features/positions/screens/PositionSheet.tsx:81` |
| `Opened position Sheet` | event | implemented | `src/features/positions/components/PositionCard.tsx:78` |
| `ownedAccounts` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `pair_hw_wallet_nav.entered` | event | implemented | `src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:39`, `src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:42` |
| `pair_hw_wallet_nav.exited` | event | implemented | `src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:39`, `src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:42` |
| `pending_transaction.resolved` | event | implemented | `src/hooks/useWatchPendingTxs.ts:92` |
| `Performance Wallet Initialize Time` | event | implemented | `src/performance/tracking/index.ts:86` |
| `performance.report` | event | implemented | `src/performance/tracking/index.ts:86` |
| `performance.time_to_sign` | event | implemented | `src/state/performance/performance.ts:62`, `src/state/performance/performance.ts:69` |
| `performance.time_to_sign.operation` | event | implemented | `src/state/performance/performance.ts:62`, `src/state/performance/performance.ts:69` |
| `perps.added_to_position` | event | implemented | `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242` |
| `perps.added_to_position.failed` | event | implemented | `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242` |
| `perps.close_position.failed` | event | implemented | `src/features/perps/screens/ClosePositionBottomSheet.tsx:150`, `src/features/perps/screens/ClosePositionBottomSheet.tsx:179` |
| `perps.closed_position` | event | implemented | `src/features/perps/screens/ClosePositionBottomSheet.tsx:150`, `src/features/perps/screens/ClosePositionBottomSheet.tsx:179` |
| `perps.open_position.canceled` | event | implemented | `src/features/perps/components/PerpsNavigatorFooter.tsx:237`, `src/features/perps/components/PerpsNavigatorFooter.tsx:249`, `src/features/perps/components/PerpsNavigatorFooter.tsx:266` and 3 more |
| `perps.open_position.failed` | event | implemented | `src/features/perps/components/PerpsNavigatorFooter.tsx:237`, `src/features/perps/components/PerpsNavigatorFooter.tsx:249`, `src/features/perps/components/PerpsNavigatorFooter.tsx:266` |
| `perps.opened_position` | event | implemented | `src/features/perps/components/PerpsNavigatorFooter.tsx:237`, `src/features/perps/components/PerpsNavigatorFooter.tsx:249`, `src/features/perps/components/PerpsNavigatorFooter.tsx:266` |
| `perps.trade_details.share_pressed` | event | implemented | `src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73` |
| `perps.trigger_order_cancel.failed` | event | implemented | `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71` |
| `perps.trigger_order_canceled` | event | implemented | `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71` |
| `perps.trigger_order_created` | event | implemented | `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268`, `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281` |
| `perps.trigger_order.failed` | event | implemented | `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268`, `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281` |
| `perps.withdraw.failed` | event | implemented | `src/features/perps/withdrawalConfig.ts:48`, `src/features/perps/withdrawalConfig.ts:57` |
| `perps.withdrew` | event | implemented | `src/features/perps/withdrawalConfig.ts:48`, `src/features/perps/withdrawalConfig.ts:57` |
| `positions.error` | event | implemented | `src/features/positions/stores/trackPositionsResponseErrors.ts:43` |
| `positionsAmount` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `positionsAssetsAmount` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `positionsDappsAmount` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `positionsRewardsAmount` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `positionsRewardsUSDValue` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `positionsUSDValue` | user property | implemented | `src/features/positions/stores/positionsStore.ts:167` |
| `predictions.deposit` | event | implemented | `src/features/polymarket/depositConfig.ts:59`, `src/features/polymarket/depositConfig.ts:60` |
| `predictions.deposit.failed` | event | implemented | `src/features/polymarket/depositConfig.ts:59`, `src/features/polymarket/depositConfig.ts:60` |
| `predictions.order_match.failed` | event | implemented | `src/features/polymarket/utils/executePolymarketOrder.ts:324`, `src/features/polymarket/utils/executePolymarketOrder.ts:353` |
| `predictions.place_order` | event | implemented | `src/features/polymarket/utils/executePolymarketOrder.ts:324`, `src/features/polymarket/utils/executePolymarketOrder.ts:353` |
| `predictions.place_order.failed` | event | implemented | `src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:209`, `src/features/polymarket/screens/polymarket-sell-position-sheet/PolymarketSellPositionSheet.tsx:127` |
| `predictions.redeem_position` | event | implemented | `src/features/polymarket/utils/redeemPosition.ts:59`, `src/features/polymarket/utils/redeemPosition.ts:82` |
| `predictions.redeem_position.failed` | event | implemented | `src/features/polymarket/utils/redeemPosition.ts:59`, `src/features/polymarket/utils/redeemPosition.ts:82` |
| `predictions.withdrawal` | event | implemented | `src/features/polymarket/withdrawalConfig.ts:62`, `src/features/polymarket/withdrawalConfig.ts:63` |
| `predictions.withdrawal.failed` | event | implemented | `src/features/polymarket/withdrawalConfig.ts:62`, `src/features/polymarket/withdrawalConfig.ts:63` |
| `Pressed Button` | event | implemented | `src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23`, `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267` and 6 more |
| `privateKeys` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `qr_code.viewed` | event | implemented | `src/components/cards/ReceiveAssetsCard.tsx:47` |
| `Rap completed` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `Rap failed` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `Rap started` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `Received wc connection` | event | implemented | `src/screens/WalletConnectApprovalSheet.tsx:260`, `src/screens/WalletConnectApprovalSheet.tsx:298` |
| `recoveryPhrases` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `refresh_account_data` | event | implemented | `src/hooks/useRefreshAccountData.ts:59` |
| `Rejected new WalletConnect session` | event | implemented | `src/features/wallet-connect/handlers/onSessionProposal.ts:38`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:158` and 1 more |
| `request.approved` | event | implemented | `src/features/dapp-request/screens/SignTransactionSheet.tsx:295`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:406`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:511` |
| `request.rejected` | event | implemented | `src/features/dapp-request/screens/SignTransactionSheet.tsx:295`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:406`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:511` |
| `request.sheet.show` | event | implemented | `src/features/dapp-request/hooks/useTransactionSetup.ts:55` |
| `Reset Asset Selection Send` | event | implemented | `src/features/transfer/screens/SendSheet.tsx:357`, `src/features/transfer/screens/SendSheet.tsx:378`, `src/features/transfer/screens/SendSheet.tsx:554` and 1 more |
| `rewards.pressed_available_card` | event | implemented | `src/screens/rewards/components/RewardsClaimed.tsx:87` |
| `rewards.pressed_bridged_card` | event | implemented | `src/screens/rewards/components/RewardsStats.tsx:34`, `src/screens/rewards/components/RewardsStats.tsx:42` |
| `rewards.pressed_leaderboard_item` | event | implemented | `src/screens/rewards/components/RewardsLeaderboardItem.tsx:81` |
| `rewards.pressed_pending_earnings_card` | event | implemented | `src/screens/rewards/components/RewardsEarnings.tsx:71` |
| `rewards.pressed_swapped_card` | event | implemented | `src/screens/rewards/components/RewardsStats.tsx:34`, `src/screens/rewards/components/RewardsStats.tsx:42` |
| `rewards.viewed_sheet` | event | implemented | `src/screens/rewards/RewardsSheet.tsx:45` |
| `rnbw_airdrop.claim` | event | implemented | `src/features/rnbw-rewards/utils/claimAirdrop.ts:49`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:91`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:116` |
| `rnbw_airdrop.claim.failed` | event | implemented | `src/features/rnbw-rewards/utils/claimAirdrop.ts:49`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:91`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:116` |
| `rnbw_rewards.claim` | event | implemented | `src/features/rnbw-rewards/utils/claimRewards.ts:75`, `src/features/rnbw-rewards/utils/claimRewards.ts:123`, `src/features/rnbw-rewards/utils/claimRewards.ts:151` |
| `rnbw_rewards.claim.failed` | event | implemented | `src/features/rnbw-rewards/utils/claimRewards.ts:75`, `src/features/rnbw-rewards/utils/claimRewards.ts:123`, `src/features/rnbw-rewards/utils/claimRewards.ts:151` |
| `rnbw_staking.stake` | event | implemented | `src/features/rnbw-staking/utils/stakeRnbw.ts:194`, `src/features/rnbw-staking/utils/stakeRnbw.ts:213` |
| `rnbw_staking.stake.failed` | event | implemented | `src/features/rnbw-staking/utils/stakeRnbw.ts:194`, `src/features/rnbw-staking/utils/stakeRnbw.ts:213` |
| `rnbw_staking.unstake` | event | implemented | `src/features/rnbw-staking/utils/unstakeRnbw.ts:73`, `src/features/rnbw-staking/utils/unstakeRnbw.ts:111`, `src/features/rnbw-staking/utils/unstakeRnbw.ts:119` |
| `rnbw_staking.unstake.failed` | event | implemented | `src/features/rnbw-staking/utils/unstakeRnbw.ts:73`, `src/features/rnbw-staking/utils/unstakeRnbw.ts:111`, `src/features/rnbw-staking/utils/unstakeRnbw.ts:119` |
| `rnbwMembershipTier` | user property | implemented | `src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:41` |
| `Saved backup password on iCloud` | event | implemented | `src/features/backup/backup.ts:187`, `src/features/backup/backup.ts:532`, `src/features/backup/backup.ts:535` |
| `Scanned address QR code` | event | implemented | `src/hooks/useScanner.ts:57`, `src/hooks/useScanner.ts:78`, `src/hooks/useScanner.ts:105` and 1 more |
| `Scanned broken or unsupported QR code` | event | implemented | `src/hooks/useScanner.ts:57`, `src/hooks/useScanner.ts:78`, `src/hooks/useScanner.ts:105` and 1 more |
| `Scanned Rainbow profile url` | event | implemented | `src/hooks/useScanner.ts:57`, `src/hooks/useScanner.ts:78`, `src/hooks/useScanner.ts:105` and 1 more |
| `Scanned WalletConnect QR code` | event | implemented | `src/hooks/useScanner.ts:57`, `src/hooks/useScanner.ts:78`, `src/hooks/useScanner.ts:105` and 1 more |
| `screenHeight` | user property | implemented | `src/App.tsx:197` |
| `screenScale` | user property | implemented | `src/App.tsx:197` |
| `screenWidth` | user property | implemented | `src/App.tsx:197` |
| `Search Query` | event | implemented | `src/components/Discover/DiscoverSearchInput.tsx:134` |
| `Sent transaction` | event | implemented | `src/features/transfer/utils/trackSentTransaction.ts:32` |
| `Set App Icon` | event | implemented | `src/features/app-icon/components/AppIconSettingsSection.tsx:32` |
| `Show Secret View` | event | implemented | `src/screens/SettingsSheet/components/Backups/ShowSecretView.tsx:10` |
| `Show wallet profile modal for ENS address` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `Show wallet profile modal for imported wallet` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `Show wallet profile modal for read only wallet` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `Show wallet profile modal for Unstoppable address` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `Showing Walletconnect signing request` | event | implemented | `src/features/wallet-connect/handlers/onSessionRequest.ts:84`, `src/features/wallet-connect/handlers/onSessionRequest.ts:119`, `src/features/wallet-connect/handlers/onSessionRequest.ts:147` and 2 more |
| `Shown Walletconnect session request` | event | implemented | `src/screens/WalletConnectApprovalSheet.tsx:260`, `src/screens/WalletConnectApprovalSheet.tsx:298` |
| `spot_pnl.pill.shown` | event | implemented | `src/screens/expandedAssetSheet/components/sections/BalanceSection.tsx:61` |
| `spot_pnl.sheet.viewed` | event | implemented | `src/features/spot-pnl/ui/SpotPnlSheet.tsx:257` |
| `stakedRnbw` | user property | implemented | `src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:41` |
| `surface.interaction` | event | implemented | `src/features/placements/engagement/trackInteraction.ts:20`, `src/features/placements/engagement/trackInteraction.ts:43` |
| `swaps.changed_chain_id` | event | implemented | `src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:75` |
| `swaps.failed` | event | implemented | `src/__swaps__/screens/Swap/providers/swap-provider.tsx:342`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:396`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023` |
| `swaps.flipped_assets` | event | implemented | `src/__swaps__/screens/Swap/components/FlipButton.tsx:97` |
| `swaps.quote_failed` | event | implemented | `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:8` |
| `swaps.received_quote` | event | implemented | `src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438` |
| `swaps.searched_for_token` | event | implemented | `src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:173`, `src/__swaps__/screens/Swap/components/TokenList/TokenToSellList.tsx:88` |
| `swaps.selected_asset` | event | implemented | `src/__swaps__/screens/Swap/providers/swap-provider.tsx:342`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:396`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023` |
| `swaps.submitted` | event | implemented | `src/__swaps__/screens/Swap/providers/swap-provider.tsx:342`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:396`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023` |
| `swaps.toggled_degen_mode` | event | implemented | `src/__swaps__/screens/Swap/hooks/useSwapSettings.ts:14` |
| `Tapped "Add Cash"` | event | implemented | `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174` |
| `Tapped "Cancel" after editing wallet` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped "Cancel" on Wallet Profile modal` | event | implemented | `src/components/expanded-state/WalletProfileState.js:54`, `src/components/expanded-state/WalletProfileState.js:61` |
| `Tapped "Confirm Backup"` | event | implemented | `src/features/backup/hooks/useCreateBackup.ts:65`, `src/features/backup/hooks/useCreateBackup.ts:92` |
| `Tapped "Delete Wallet" (final confirm)` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped "Done" after editing wallet` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped "Edit Wallet"` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped "Get a new wallet"` | event | implemented | `src/screens/WelcomeScreen/WelcomeScreen.tsx:163`, `src/screens/WelcomeScreen/WelcomeScreen.tsx:201` |
| `Tapped "I already have one"` | event | implemented | `src/screens/WelcomeScreen/WelcomeScreen.tsx:163`, `src/screens/WelcomeScreen/WelcomeScreen.tsx:201` |
| `Tapped "Import" button` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `Tapped "My QR Code"` | event | implemented | `src/components/asset-list/RecyclerAssetList2/index.tsx:102` |
| `Tapped "Notification Settings"` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped "Send"` | event | implemented | `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174` |
| `Tapped "Submit" on Wallet Profile modal` | event | implemented | `src/components/expanded-state/WalletProfileState.js:54`, `src/components/expanded-state/WalletProfileState.js:61` |
| `Tapped "Swap"` | event | implemented | `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174` |
| `Tapped Add Existing Wallet` | event | implemented | `src/screens/AddWalletSheet.tsx:45`, `src/screens/AddWalletSheet.tsx:49`, `src/screens/AddWalletSheet.tsx:58` and 5 more |
| `Tapped Create New Wallet` | event | implemented | `src/screens/AddWalletSheet.tsx:45`, `src/screens/AddWalletSheet.tsx:49`, `src/screens/AddWalletSheet.tsx:58` and 5 more |
| `Tapped Delete Wallet` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped Edit` | event | implemented | `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`, `src/screens/change-wallet/ChangeWalletSheet.tsx:286` and 5 more |
| `Tapped Push Notification` | event | implemented | `src/notifications/analytics.ts:11`, `src/notifications/analytics.ts:17` |
| `Tapped Search` | event | implemented | `src/components/Discover/DiscoverScreenContext.tsx:64` |
| `Tapped Watch Address` | event | implemented | `src/screens/AddWalletSheet.tsx:45`, `src/screens/AddWalletSheet.tsx:49`, `src/screens/AddWalletSheet.tsx:58` and 5 more |
| `Time spent on the Discover screen` | event | implemented | `src/components/Discover/useTrackDiscoverScreenTime.ts:22` |
| `Toggled an NFT as Hidden` | event | implemented | `src/hooks/useHiddenTokens.ts:34`, `src/hooks/useHiddenTokens.ts:73` |
| `token_details.nft` | event | implemented | `src/components/expanded-state/UniqueTokenExpandedState.tsx:389` |
| `token_launcher.abandoned` | event | implemented | `src/screens/token-launcher/components/TokenLauncherHeader.tsx:59` |
| `token_launcher.creation_failed` | event | implemented | `src/screens/token-launcher/state/tokenLauncherStore.ts:292`, `src/screens/token-launcher/state/tokenLauncherStore.ts:379`, `src/screens/token-launcher/state/tokenLauncherStore.ts:438` |
| `token_launcher.image_upload_failed` | event | implemented | `src/screens/token-launcher/hooks/useUploadToCloudinary.ts:29` |
| `token_launcher.share_pressed` | event | implemented | `src/screens/token-launcher/components/TokenLauncherFooter.tsx:61`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:79`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:120` and 1 more |
| `token_launcher.step_changed` | event | implemented | `src/screens/token-launcher/state/tokenLauncherStore.ts:292`, `src/screens/token-launcher/state/tokenLauncherStore.ts:379`, `src/screens/token-launcher/state/tokenLauncherStore.ts:438` |
| `token_launcher.token_created` | event | implemented | `src/screens/token-launcher/state/tokenLauncherStore.ts:292`, `src/screens/token-launcher/state/tokenLauncherStore.ts:379`, `src/screens/token-launcher/state/tokenLauncherStore.ts:438` |
| `token_launcher.wallet_load_failed` | event | implemented | `src/screens/token-launcher/components/TokenLauncherFooter.tsx:61`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:79`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:120` and 1 more |
| `token_list` | event | implemented | `src/__swaps__/screens/Swap/hooks/useSearchCurrencyLists.ts:161`, `src/components/Discover/DiscoverSearch.tsx:117`, `src/features/transfer/screens/SendSheet.tsx:357` and 4 more |
| `top_holders.shown` | event | implemented | `src/screens/expandedAssetSheet/components/sections/TopHoldersSection.tsx:44` |
| `Transferred ENS control` | event | implemented | `src/features/ens/raps/actions.ts:394`, `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:422` and 9 more |
| `trezorDevices` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `Updated Gas Price` | event | implemented | `src/features/gas/redux/gas.ts:488` |
| `View how to Enable iCloud` | event | implemented | `src/hooks/useWalletCloudBackup.ts:56`, `src/hooks/useWalletCloudBackup.ts:66`, `src/hooks/useWalletCloudBackup.ts:74` and 2 more |
| `Viewed ENS profile` | event | implemented | `src/hooks/useOnAvatarPress.ts:111` |
| `Viewed external dapp` | event | implemented | `src/features/positions/screens/PositionSheet.tsx:68`, `src/features/positions/screens/PositionSheet.tsx:81` |
| `Viewed profile` | event | implemented | `src/screens/ProfileSheet.tsx:68` |
| `wallet_create.failed` | event | implemented | `src/model/wallet.ts:450`, `src/model/wallet.ts:498`, `src/model/wallet.ts:506` and 2 more |
| `wallet_create.succeeded` | event | implemented | `src/model/wallet.ts:450`, `src/model/wallet.ts:498`, `src/model/wallet.ts:506` and 2 more |
| `wallet_initialization.failed` | event | implemented | `src/state/wallets/initializeWallet.ts:190` |
| `watch_wallet` | event | implemented | `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170` and 7 more |
| `watchedAccounts` | user property | implemented | `src/hooks/useWalletCohort.ts:64` |
| `wc.failed_request` | event | implemented | `src/features/wallet-connect/handlers/onSessionProposal.ts:38`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:158` and 7 more |

## Findings

### F1. Raw wallet addresses and ENS names sent as event properties, bypassing the wallet-address hash
**High.** _Confirmed by the verifier._
The wrapper attaches only `walletAddressHash` to every event, so the design avoids sending plain addresses. These import-flow events send the user's plain address, ENS or Unstoppable name (`address`, `addressOrEnsName`, `input`, `ens`) as event properties. `input` can be whatever the user typed into the import field. When the user pasted a seed phrase, `input` is the sanitized seed phrase, but the plain-address events in this file (watchWallet, showWalletProfileModalForReadOnlyWallet) only reach it for address inputs. These properties tie an analytics profile to a public identity and to watched wallets. The values are also unbounded and have high cardinality.
- **Evidence:** `src/hooks/useImportingWallet.ts:163`, `src/hooks/useImportingWallet.ts:170`, `src/hooks/useImportingWallet.ts:200`, `src/hooks/useImportingWallet.ts:222`, `src/hooks/useImportingWallet.ts:233`, `src/hooks/useImportingWallet.ts:279`
- **Fix:** Stop sending address, ENS name and raw input. Send a hashed address, a boolean such as `is_ens` or `input_type = ens | unstoppable | address | seed`, and the existing `type`. Dropping the properties changes the existing events' schema, so add the properties as new fields or events, deprecate the old ones with `replaced_by`, and dual-fire for one release.
- **Items:** `watchWallet`, `showWalletProfileModalForENSAddress`, `showWalletProfileModalForUnstoppableAddress`, `showWalletProfileModalForReadOnlyWallet`, `showWalletProfileModalForImportedWallet`
- **Verifier:** useImportingWallet.ts:163-173, 193-203, 222-225, 233-236, 271-274, 279-282 - These calls send the plain address, the ENS or Unstoppable name, and the typed `input`, even though the wrapper already adds a hash. One correction: watchWallet is not limited to address inputs. In the derive branch, line 273 sends `address: input`, which can be a seed phrase (see F5).

### F2. Raw token-search text is sent as an event property
**High.** _Confirmed by the verifier._
The raw text the user typed into the swap and Discover token searches is sent as `query`. The swap token lists send the full `searchQuery` on `swapsSearchedForToken`. `tokenList` sends `query` from both the swap search hook and Discover search. The Discover search input sends `query` together with `length` on `searchQuery`. Search text is free user input and can contain pasted addresses or ENS names, which can be linked to a wallet because every event also carries `walletAddressHash`. The design rules say to send `char_count` and `result_count`, never the query. `length` is already sent next to the raw query on `searchQuery`.
- **Evidence:** `src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:173`, `src/__swaps__/screens/Swap/components/TokenList/TokenToSellList.tsx:88`, `src/__swaps__/screens/Swap/hooks/useSearchCurrencyLists.ts:155`, `src/components/Discover/DiscoverSearch.tsx:111`, `src/components/Discover/DiscoverSearchInput.tsx:134`
- **Fix:** Drop `query` from all three events and send `char_count` plus `result_count`. If a query is needed to debug empty results, send only a flag such as `looks_like_address`. This removes a property from existing events, so give it a SEMANTICS changelog line. The property is additive removal and does not break the event names.
- **Items:** `swaps.searched_for_token`, `token_list`, `Search Query`
- **Verifier:** DiscoverSearch.tsx:111 sends `query: searchQueryForSearch`. The registry also documents `query` on swaps.searched_for_token and on Search Query - The raw search text is sent, and `length` already sits next to it on Search Query.

### F3. Free-text and user-entered values are sent as event properties
**High.** _Confirmed by the verifier._
`tappedDoneEditingWallet` sends the label the user typed for their wallet as `wallet_label`. `viewedProfile` sends the ENS name of the viewed profile as `name`, which identifies another person's wallet. `claimClaimableFailed` sends the raw `e.message` from the provider or signer as `errorMessage`. That text is unbounded and can contain addresses, calldata or RPC URLs. The design rules say never to send raw user text or identity values as event properties. Send lengths, booleans or enum codes instead.
- **Evidence:** `src/screens/change-wallet/ChangeWalletSheet.tsx:264`, `src/screens/ProfileSheet.tsx:68`, `src/screens/ProfileSheet.tsx:71`, `src/screens/Airdrops/utils.ts:155`
- **Fix:** Replace `wallet_label` with `has_label` or `char_count`. Replace the ENS `name` with `is_own_profile` or `has_ens`. Replace `errorMessage` with a mapped `failure_reason` enum. Removing or renaming these properties changes existing events: add the new property alongside, stop sending the old one after a dual-send window, and mark the old property deprecated in the registry.
- **Items:** `claimClaimableFailed`, `tappedDoneEditingWallet`, `viewedProfile`
- **Verifier:** ChangeWalletSheet.tsx:264 sends `wallet_label: name`, ProfileSheet.tsx:71 sends the ENS `name`, and Airdrops/utils.ts:139/155 sends the raw `e.message` as errorMessage - All three are free text or identity values.

### F4. swaps.submitted also fires after a 'handled' failure, so one attempt can count as failed and submitted
**High.** _Confirmed by the verifier._
When `errorMessage` is set, `swapsFailed` is tracked. The branch only returns for non-'handled' errors, so for 'handled' the code falls through to the recent-swap and sheet-dismissal code. It then reaches `trackSwapEvent(swapsSubmitted)`. The swap funnel, the submitted count and the failure rate are all affected. The registry description flags this as unconfirmed, and the code confirms it. On this path `swaps.submitted` carries `errorMessage = null`, which is wrong for that attempt.
- **Evidence:** `src/__swaps__/screens/Swap/providers/swap-provider.tsx:342`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:349`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:396`
- **Fix:** Return after tracking `swapsFailed` in the 'handled' case, or only track submitted when `errorMessage` is falsy. This changes the firing moment of a core money event, so add a `SEMANTICS:` changelog line with the date. Tell analysts that counts on 'handled' errors will drop.
- **Items:** `swaps.submitted`, `swaps.failed`
- **Verifier:** swap-provider.tsx:337-362. The `return` only happens inside `errorMessage !== 'handled'`, so a 'handled' error falls through to line 396 and sends swapsSubmitted. trackSwapEvent.ts:49 then sets errorMessage to null.

### F5. Raw input (possibly a seed phrase or private key) is sent as `address` on watch_wallet
**High.** _Confirmed by the verifier._
In the last branch of handlePressImportButton, the input is not an ENS name, an Unstoppable domain or a valid 0x address. It is whatever deriveAccountFromWalletInput accepts, which includes seed phrases and private keys. The watch_wallet call sends `address: input`, the raw sanitized text, even though the derived `walletResult.address` is available. If someone pastes a secret into the watch flow, the secret goes to analytics. The registry description for `address` ('the raw input otherwise') confirms this path.
- **Evidence:** `src/hooks/useImportingWallet.ts:247`, `src/hooks/useImportingWallet.ts:271`, `src/hooks/useImportingWallet.ts:273`
- **Fix:** In this branch send `walletResult.address` (or nothing) instead of `input`. Never send raw input. This changes the meaning of `address` on the existing event, so add a `SEMANTICS:` changelog line. The change only removes secret material, so no dual-fire is needed.
- **Items:** `watch_wallet`
- **Verifier:** useImportingWallet.ts:245-274. The last branch accepts anything deriveAccountFromWalletInput takes, and with type 'watch' it sends `address: input`, the sanitized raw text, even though `walletResult.address` is available. Whether the watch screen lets someone paste a secret is a UI question, but the code path exists.

### F6. Raw decoded QR content is sent as an event property
**High.** _Confirmed by the verifier._
handleScanInvalid sends the full decoded text of any QR code the app can't recognise as `qrCodeData`. This event fires only for content the app doesn't understand, so it is arbitrary third-party content. It could be a URL with tokens, contact details, a payment request, or a QR code of someone's seed phrase or private key. The design rules say never to send raw user text. Send lengths or buckets instead.
- **Evidence:** `src/hooks/useScanner.ts:124`, `src/analytics/event.ts:1042`
- **Fix:** Stop sending the raw string. Send `char_count` and a coarse `content_kind` bucket instead (for example url, hex_string, other, computed locally). Optionally add `error_reason`. Removing `qrCodeData` changes the event's schema, but it only drops data. Keep the event name, and mark `qrCodeData` as deprecated in the registry. Also confirm that the analytics project does not already hold values that need purging.
- **Items:** `Scanned broken or unsupported QR code`
- **Verifier:** The registry entry and event type show that handleScanInvalid sends `qrCodeData`, the full decoded payload, for any unrecognised QR code.

### F7. User-typed wallet group name is sent as an event property
**High.** _Confirmed by the verifier._
On submit, `addNewWalletGroupName` sends `name: value.trim()`, the free text the user typed into the wallet-group input. Raw user text is on the 'never send' list. People name groups after themselves, family members or institutions. The event schema declares `name: string`, so this is deliberate rather than an accident.
- **Evidence:** `src/components/expanded-state/NewWalletGroupState.tsx:20`, `src/components/expanded-state/NewWalletGroupState.tsx:21`, `src/analytics/event.ts:732`
- **Fix:** Drop `name` and send `char_count` instead, or send nothing, since the question is only how many groups get created. Changing the schema alters an existing event, so keep the event name. Note in the registry that older rows carry names, and consider purging them from the analytics tool.
- **Items:** `add_new_wallet_group.name`
- **Verifier:** The registry entry and the cited lines show `name: value.trim()`, raw user text, sent on submit.

### F8. Money and on-chain outcomes are split into separate `*Failed` events instead of one event with `status`
**Medium.** _Confirmed by the verifier._
Each money flow (place order, deposit, withdrawal, redeem, rewards claim, airdrop claim, stake, unstake) has a success event and a separate `_failed` event. Conversion rates and funnels need a union of the two, and a funnel on the success event silently drops the failures. The failure events carry free-text `errorMessage` (`error.message`) and no normalised `failure_reason`, so failures can't be grouped. In the airdrop and rewards claim success events, `status` is the backend's final status and not a fixed success/failed value. Order failures are also tracked in two places. The sheet's catch block fires `predictionsPlaceOrderFailed` (buy and sell), and the executor separately fires `predictionsOrderMatchFailed`. These two events may describe the same failed order.
- **Evidence:** `src/features/polymarket/utils/executePolymarketOrder.ts:324`, `src/features/polymarket/utils/executePolymarketOrder.ts:353`, `src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:209`, `src/features/polymarket/screens/polymarket-sell-position-sheet/PolymarketSellPositionSheet.tsx:127`, `src/features/polymarket/utils/redeemPosition.ts:82`, `src/features/polymarket/withdrawalConfig.ts:62`, `src/features/rnbw-rewards/utils/claimRewards.ts:151`, `src/features/rnbw-staking/utils/stakeRnbw.ts:213`
- **Fix:** Don't rename in place. Add `status = success | failed | cancelled` and a normalised `failure_reason` to the outcome events. Keep the `_failed` events during a dual-fire window, then mark them `deprecated` with `replaced_by`. Write a `SEMANTICS:` changelog line, because money events are core and sit on dashboards. Replace `errorMessage` with a bounded `failure_reason` enum. Decide whether `predictionsOrderMatchFailed` is a separate stage or a duplicate of the order failure, and document it.
- **Items:** `predictionsPlaceOrder`, `predictionsPlaceOrderFailed`, `predictionsOrderMatchFailed`, `predictionsDeposit`, `predictionsDepositFailed`, `predictionsWithdrawal`, `predictionsWithdrawalFailed`, `predictionsRedeemPosition`, `predictionsRedeemPositionFailed`, `rnbwRewardsClaim`, `rnbwRewardsClaimFailed`, `rnbwAirdropClaim`, `rnbwAirdropClaimFailed`, `rnbwStakingStake`, `rnbwStakingStakeFailed`, `rnbwStakingUnstake`, `rnbwStakingUnstakeFailed`
- **Verifier:** The registry has separate success and `.failed` entries for place_order, redeem, withdraw, rnbw claims, stake and unstake. The failed events carry free-text errorMessage, and PolymarketNewPositionSheet.tsx:209 sends place_order.failed separately from order_match.failed - This is the rule-4 split.

### F9. Cash outcomes are split into separate *_Failed events instead of a status on one outcome event
**Medium.** _Confirmed by the verifier._
Each flow sends a success event and a separate failure event. Examples: buy order completed/failed, KYC approved/failed, card linked/link-failed, phone verified/verify-failed, sign-in succeeded/failed/cancelled. Every conversion rate therefore needs a union of events, and funnels drop the failures. Rule 4 says failure belongs on the outcome event as `status` (success, failed, cancelled) plus `failure_reason`. The failure properties are also inconsistent. `cashBuyOrderFailed` sends `failureReason` and `errorCode` in camelCase. The KYC, phone and card events send `reason`. Sign-in cancellation is already a separate event, which fits the `cancelled` status value.
- **Evidence:** `src/features/cash/stores/cashBuyOrderStore.ts:146`, `src/features/cash/stores/cashBuyOrderStore.ts:233`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:231`, `src/features/cash/stores/cardLinkFlowStore.ts:58`, `src/features/cash/stores/verifyPhoneFlowStore.ts:155`, `src/features/cash/services/cashSignInService.ts:76`, `src/features/cash/services/cashSignInService.ts:78`
- **Fix:** Add the outcome events with `status` and `failure_reason`, and use one property vocabulary. Don't rename in place. Deprecate the old `*Failed` events with `replaced_by`, and dual-fire the money events (buy order) and KYC events for one release (`dual_fire_until`). Add a `SEMANTICS:` changelog line.
- **Items:** `cashBuyOrderFailed`, `cashBuyOrderCompleted`, `cashKycFailed`, `cashKycApproved`, `cashCardLinkFailed`, `cashCardLinked`, `cashPhoneVerifyFailed`, `cashPhoneVerified`, `cashSignInFailed`, `cashSignInSucceeded`, `cashSignInCancelled`, `cashPhoneSubmitFailed`
- **Verifier:** cashBuyOrderStore.ts:123 and :146 are separate completed and failed events. The failure properties are named inconsistently: `failureReason`/`errorCode` here, `reason` on the other cash events.

### F10. Failure events are split from outcomes and miss several failure and cancel paths in the cash flows
**Medium.** _Confirmed by the verifier._
Each cash step has a success event and a separate `*_failed` event, instead of one outcome event with `status` and `failure_reason`. Every conversion rate then needs a union of events. The failure events also fire only on some paths. In `useAddPasskeyFlow.submit`, access refusal, passkey cancellation and 'already on device' all return before `cashPasskeyFailed`, and there is no `cancelled` signal. In `useWalletLinkFlow`, `WalletSignatureError` and handled errors return silently. In `advanceToChallenge`, access refusal returns without tracking. Funnels from submitted to added therefore show drop-off that is really cancellation or refusal, and the failure counts understate what happened. The same code logs through `logger.error` on exactly the paths that do get an event.
- **Evidence:** `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:60`, `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:73`, `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:42`, `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:47`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78`
- **Fix:** Move to one outcome event per step (for example `cash_passkey_completed`) with `status = success | failed | cancelled` and `failure_reason`. Treat cancelled as not a failure. This is a rename, so go through deprecate + `replaced_by` and dual-fire the old events for one release, with cut-over by app version. Until then, document in the registry which paths emit no event.
- **Items:** `cash.passkey_failed`, `cash.phone_submit_failed`, `cash.wallet_link_failed`, `cash.wallet_check_failed`, `cash.passkey_added`, `cash.phone_submitted`, `cash.wallet_linked`
- **Verifier:** useAddPasskeyFlow.ts:59-71. A cash-access refusal, a passkey cancellation and a passkey already on the device all return before cashPasskeyFailed (line 73), so those paths send no event.

### F11. Backup events: 'skipped password' fires on any keychain error, and event names are inconsistent
**Medium.** _Confirmed by the verifier._
`saveBackupPassword` tracks 'Didn't save backup password' in the catch block, so any keychain failure is counted as the user skipping. A skip and a failure are different things. Backup success and failure are also split across `Backup Complete` and `backup.error`. The names mix three styles: Title Case (`Backup Complete`), free text with quotes (`Tapped "Confirm Backup"`) and dotted lower case (`backup.error`). Breakdowns and searches by name are awkward, and the generic `category` and `label` properties repeat constants.
- **Evidence:** `src/features/backup/backup.ts:532`, `src/features/backup/backup.ts:535`, `src/features/backup/backup.ts:187`, `src/features/backup/hooks/useCreateBackup.ts:65`, `src/features/backup/hooks/useCreateBackup.ts:92`, `src/analytics/event.ts:271`
- **Fix:** Add a `status` or `failure_reason` to the password event, or track the catch separately as a failure. Don't rename in place. Introduce `object_action` snake_case events, deprecate the old names with `replaced_by`, and dual-fire `Backup Complete` for one release because it is a core event.
- **Items:** `backup.error`, `Saved backup password on iCloud`, `Didn't save backup password on iCloud`, `Backup Complete`, `Tapped "Confirm Backup"`, `backup.sheet_shown`
- **Verifier:** backup.ts:529-536. Any throw from setSharedWebCredentials sends backupSkippedPassword with no reason, so failures are counted as skips.

### F12. Perps failures are separate `_failed` events instead of `status`/`failure_reason` on the outcome event, and carry raw error text
**Medium.** _Confirmed by the verifier._
Close position, create trigger order and cancel trigger order each emit one success event and a separate failure event from the catch branch. Every conversion rate for these money-adjacent flows needs a union of two events, and funnels drop the failures. The failure events send `errorMessage`, which is the free-text string from `parseHyperliquidErrorMessage`, not a bounded enum, so it can't be broken down cleanly.
- **Evidence:** `src/features/perps/screens/ClosePositionBottomSheet.tsx:150`, `src/features/perps/screens/ClosePositionBottomSheet.tsx:179`, `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268`, `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71`
- **Fix:** Add `status = success | failed` and a bounded `failure_reason` to the outcome events (`perps_position_closed`, `perps_trigger_order_created`, `perps_trigger_order_canceled`). Map the Hyperliquid error to an enum and mark values you haven't seen `(UNCONFIRMED)`. Don't rename in place: keep the old `*Failed` events as deprecated with `replaced_by` and dual-fire for one release, because they sit on perps dashboards.
- **Items:** `perpsClosePositionFailed`, `perpsClosedPosition`, `perpsTriggerOrderFailed`, `perpsTriggerOrderCreated`, `perpsTriggerOrderCancelFailed`, `perpsTriggerOrderCanceled`
- **Verifier:** The registry shows separate perps `*.failed` events carrying the output of parseHyperliquidErrorMessage. The close and cancel calls I read (ClosePositionBottomSheet.tsx:179, TriggerOrdersSection.tsx:71) match.

### F13. token_created fires before the rest of the launch can fail, so one launch can count as both created and failed
**Medium.** _Confirmed by the verifier._
token_created is sent as soon as launchToken returns, but the same try block then awaits publicClient.getTransaction. If that call throws, the catch branch sends token_launcher.creation_failed for a coin that was created. Failure is also modelled as three separate events (creation_failed, image_upload_failed, wallet_load_failed) instead of status and failure_reason on one outcome event (rule R004). Any launch success rate then needs a union, and the double-fired launches inflate both sides. share_pressed also fires after the share sheet closes whether or not the person shared, so it counts opened sheets, not shares.
- **Evidence:** `src/screens/token-launcher/state/tokenLauncherStore.ts:379`, `src/screens/token-launcher/state/tokenLauncherStore.ts:386`, `src/screens/token-launcher/state/tokenLauncherStore.ts:438`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:61`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:212`
- **Fix:** Move the token_created call so it fires after the part that can still fail, or send it with status and put failures on the same outcome event with failure_reason. Because this changes the firing moment, add a changelog line starting SEMANTICS:. Add the failure form of the event first, then deprecate the _failed events with replaced_by and dual-fire for one release. Don't delete or rename them in place. Say in the share_pressed description that it fires when the sheet closes, not on a completed share.
- **Items:** `token_launcher.token_created`, `token_launcher.creation_failed`, `token_launcher.image_upload_failed`, `token_launcher.wallet_load_failed`, `token_launcher.share_pressed`
- **Verifier:** tokenLauncherStore.ts:379 tracks token_created, then lines 386 and 417 (getTransaction, addNewTransaction) run in the same try, and the catch at 438 sends creation_failed. The share_pressed point is confirmed by TokenLauncherFooter.tsx:203-212.

### F14. Wallet creation and backup failures are separate events with free-text error values
**Medium.** _Confirmed by the verifier._
Wallet creation uses a success event and a failure event. The failure event's `error` is a mix of fixed codes ('wallet_or_address_missing', 'wallet_already_exists') and raw `error.message` text. The raw text splits any breakdown into many buckets. Backup failure is split into one event per cloud platform plus a third event for status-update failure, and the platform is repeated in `label`. Every conversion rate now needs a union, and funnels drop the failures.
- **Evidence:** `src/model/wallet.ts:450`, `src/model/wallet.ts:506`, `src/model/wallet.ts:722`, `src/model/wallet.ts:730`, `src/hooks/useWalletCloudBackup.ts:131`, `src/hooks/useWalletCloudBackup.ts:152`
- **Fix:** Move to one outcome event per flow (for example wallet_created and wallet_backup_completed) with `status` and a bounded `failure_reason` enum, plus `cloud_platform` as a property. Map unknown errors to 'unknown' and log the message elsewhere. Don't rename in place. Add the new events, mark the old ones deprecated with `replaced_by`, and dual-fire for one release.
- **Items:** `walletCreateFailed`, `walletCreateSucceeded`, `errorDuringGoogleDriveBackup`, `errorDuringICloudBackup`, `errorUpdatingBackupStatus`
- **Verifier:** wallet.ts:730-732 sends the raw `error.message` on wallet_create.failed. The registry shows the per-platform backup error events (useWalletCloudBackup.ts:131-138) and the separate status-update event (:152).

### F15. A successful claim can also be reported as failed when the post-claim store refresh throws
**Medium.** _Confirmed by the verifier._
In both claim flows the post-claim balance refresh sits inside the same try as the claim, so its errors reach the catch block that sends the failed event. In claimRewards.ts the success event is sent at line 123. The user-assets and rewards-balance refreshes then run at lines 141-144. If one throws, the catch at line 151 sends rnbw_rewards.claim.failed for the same claim, so the claim is counted as both success and failure. In claimAirdrop.ts the refresh at line 90 runs before the success event at line 91. A refresh error therefore sends only the failed event, and the claim that really succeeded is never counted. Claim success rate and RNBW claimed are both distorted. The registry description for rnbw_airdrop.claim already notes this.
- **Evidence:** `src/features/rnbw-rewards/utils/claimRewards.ts:123`, `src/features/rnbw-rewards/utils/claimRewards.ts:141`, `src/features/rnbw-rewards/utils/claimRewards.ts:151`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:90`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:116`
- **Fix:** Send the success event straight after polling confirms the claim. Run the store refreshes in their own try/catch, or after the main try block, so they cannot reach the failure tracking. This changes the firing moment of rnbw_airdrop.claim, so add a `SEMANTICS:` changelog line with the date.
- **Items:** `rnbw_rewards.claim`, `rnbw_rewards.claim.failed`, `rnbw_airdrop.claim`, `rnbw_airdrop.claim.failed`
- **Verifier:** In claimRewards.ts the success event at 123 is followed by awaited refreshes at 141/144 inside the same try, with the catch at 151. In claimAirdrop.ts the awaited refresh at 90 comes before the success event at 91, with the catch at 116.

### F16. Swap events attach whole asset and quote objects as properties
**Medium.** _Confirmed by the verifier._
`swapsSelectedAsset`, `swapsReceivedQuote` and `swapsChangedChainId` pass whole `ParsedSearchAsset` and `Quote` objects. These are large nested payloads that include icons, colours, network maps and balances, and `balance` is user wallet data. `swapsReceivedQuote` fires on every quote fetch, including error responses and the repeated refresh polling, because it runs before the error check. This inflates event size and volume and produces unqueryable nested properties. Compare `analyticsTrackQuoteFailed`, which sends only address, chainId and symbol and de-duplicates repeats. `claimablePanelOpened` sends per-asset address and amount arrays, which are also hard to break down.
- **Evidence:** `src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023`, `src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:75`, `src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88`, `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:38`
- **Fix:** Flatten to scalar properties such as `input_chain_id`, `input_symbol`, `output_symbol`, `quote_status` and `error_code`. De-duplicate `swapsReceivedQuote` as `analyticsTrackQuoteFailed` does. This is a breaking change for existing dashboards. Add the new flat properties first, dual-fire for one release, then deprecate the old shape.
- **Items:** `swaps.selected_asset`, `swaps.received_quote`, `swaps.changed_chain_id`, `claimable_panel.opened`
- **Verifier:** useSwapInputsController.ts:438-442 sends whole asset and quote objects before the error check at 444. swap-provider.tsx:1023 sends whole asset objects too.

### F17. Browsing URLs and free-text names sent as event properties (favorites, trending dapps, prediction markets)
**Medium.** _Confirmed by the verifier._
`browserAddFavorite` sends the user-saved `url`, `name` and `image`. `browserTapFavorite` passes the whole favourited-site object as properties (`analytics.track(event, dapp)`), so any field later added to the type is sent without anyone deciding to. Favourite and visited URLs are user-chosen, can include arbitrary sites, and are attached to the wallet hash by the wrapper. The market name and slug are sent on every prediction order press.
- **Evidence:** `src/features/dapp-browser/stores/favoriteDappsStore.ts:50`, `src/features/dapp-browser/components/Homepage.tsx:226`, `src/features/dapp-browser/components/Homepage.tsx:334`, `src/features/discover/components/markets/layouts/MarketList.tsx:82`
- **Fix:** Send an explicit allow-list of properties. For user-chosen sites send the hostname, or a flag for curated vs. custom dapp, not the full URL or image. Only send names for curated dapps.
- **Items:** `browserAddFavorite`, `browserTapFavorite`, `browserTrendingDappClicked`, `discoverPredictionOrderPressed`
- **Verifier:** Homepage.tsx:226 calls `analytics.track(analytics.event.browserTapFavorite, dapp)` with the whole object. The registry shows browser.add_favorite sending url, name and image.

### F18. Three events answer one question about how a wallet import starts, and 'read only wallet' fires for any address
**Medium.** _Confirmed by the verifier._
The same action, starting an import or watch from a typed input, is split into one event per input format (ENS, Unstoppable, plain address, and a fourth for seed phrases). Rule 3 says this should be one event with an `input_kind` property. The 'read only wallet' event is also misnamed. It fires for any valid 0x address, including type='import', and not only watch. It fires inside the try block after the reverse ENS lookup, so a failed lookup (line 227) means it never fires, even though the flow continues to startImportProfile at line 239. The count of wallet additions that start from an address is therefore undercounted. The ENS and Unstoppable events fire only after resolution succeeds. A failed or abandoned resolution (line 153, 185) leaves no event, so there is no intent/outcome pair for drop-off.
- **Evidence:** `src/hooks/useImportingWallet.ts:209`, `src/hooks/useImportingWallet.ts:213`, `src/hooks/useImportingWallet.ts:222`, `src/hooks/useImportingWallet.ts:226`, `src/hooks/useImportingWallet.ts:239`
- **Fix:** Do not rename in place. Add one event, for example `wallet_import_started`, with `input_kind`, `type` and `status`. Mark the three old entries deprecated with `replaced_by`. Dual-fire for one release, and cut dashboards over by app version because old installs keep sending the old names. Move the read-only call out of the try block, or fire it in the catch path too.
- **Items:** `Show wallet profile modal for ENS address`, `Show wallet profile modal for Unstoppable address`, `Show wallet profile modal for read only wallet`
- **Verifier:** The rule-3 split, the firing on type 'import', and the missing intent event when ENS or Unstoppable resolution fails (lines 151-155, 183-187) all hold. One part is wrong: fetchReverseRecord (handlers.ts:839-848) never throws, so the read-only event is not undercounted and does not need moving out of the try (see F93).

### F19. `positionsUSDValue` is set to the same amount as `positionsValue`, so one of the two is mislabelled
**Medium.** _Confirmed by the verifier._
When the user's currency isn't USD, the code adds `positionsUSDValue: position.totals.total.amount`. That is the same field already sent as `positionsValue` next to `positionsCurrency: nativeCurrency`. Either `positionsValue` is really USD and its currency label is wrong, or `positionsUSDValue` holds a non-USD number. Any sum or average of these properties is wrong for non-USD users. The `identify` call in `positionsStore.ts` also writes the profile property `positionsUSDValue` from the same `totals.total.amount` field. A currency conversion is needed on one of the two, or one of the properties should be dropped.
- **Evidence:** `src/features/positions/components/PositionCard.tsx:81`, `src/features/positions/components/PositionCard.tsx:83`, `src/features/positions/screens/PositionSheet.tsx:73`, `src/features/positions/screens/PositionSheet.tsx:74`, `src/features/positions/stores/positionsStore.ts:169`
- **Fix:** Confirm which currency `position.totals.total.amount` is in. If it is USD, send it as the USD property only and drop the native-currency label, or convert for `positionsValue`. If it is in the native currency, stop sending it as `positionsUSDValue`, or convert it. Since this changes the meaning of existing properties, introduce new property names and don't change the type or unit under the old names.
- **Items:** `positionsOpenedSheet`, `positionsOpenedExternalDapp`, `positionsOpenedAsset`
- **Verifier:** PositionCard.tsx:81-83 sends `positionsUSDValue` with the same `position.totals.total.amount` as `positionsValue`, which is labelled with the native currency.

### F20. Claimables user properties are built from runtime names
**Medium.** _Confirmed by the verifier._
identify() is called with one property per claimable type, named `claimable-${claimable.type}-USDValue`. The property set therefore depends on the data, and a new claimable type creates a new profile property that is not in the plan. The names also use a different style from the camelCase used elsewhere. Values are stored as strings, so they can't be summed or averaged.
- **Evidence:** `src/resources/addys/claimables/query.ts:73`, `src/resources/addys/claimables/query.ts:83`
- **Fix:** Keep `claimablesAmount` and `claimablesUSDValue` as numbers. Drop the per-type properties, or limit them to a fixed list of types in the registry. Changing a type needs a new property name rather than a new type on the old name.
- **Items:** `claimable-<type>-USDValue`, `claimablesAmount`, `claimablesUSDValue`
- **Verifier:** claimables/query.ts:73-83 builds identify keys like `claimable-${claimable.type}-USDValue`, and the values are strings from add().

### F21. Property type mixes number and string; events fire at unreliable moments
**Medium.** _Confirmed by the verifier._
`waitingTime` on `receivedWcConnection` is a number of seconds, but it becomes the string 'Error calculating waiting time.' when `receivedTimestamp` is missing. Analytics tools split or drop a property with mixed types, so averages go wrong. `learnArticleOpened` fires in the cleanup of a mount-only effect, so it fires when the screen closes, not when the article opens. Its `durationSeconds` is correct, but the event name says 'opened' and the count is of closes. Also, the deps array contains `url`, `key` and similar values, so any change to them fires the event again and resets nothing. The WalletConnect sheet fires two separate mount effects, `shownWalletconnectSessionRequest` and `receivedWcConnection`. These effectively double-count one sheet view.
- **Evidence:** `src/screens/WalletConnectApprovalSheet.tsx:301`, `src/screens/WalletConnectApprovalSheet.tsx:260`, `src/screens/WalletConnectApprovalSheet.tsx:298`, `src/screens/LearnWebViewScreen.tsx:33`, `src/screens/LearnWebViewScreen.tsx:35`, `src/screens/LearnWebViewScreen.tsx:45`
- **Fix:** Send `waitingTime` as a number only: omit it or use null when it can't be computed. If a type change is needed, add a new property name. Rename or document `learnArticleOpened` as a close or read event (SEMANTICS note in the changelog), or move it to fire on mount and add a separate `learnArticleClosed` with `durationSeconds`. Merge or document the two WalletConnect events.
- **Items:** `receivedWcConnection`, `learnArticleOpened`, `shownWalletconnectSessionRequest`
- **Verifier:** WalletConnectApprovalSheet.tsx:301 can send the string 'Error calculating waiting time.' for waitingTime. LearnWebViewScreen.tsx:33-45 sends from a cleanup whose effect has dependencies. The effect at 258-260 sends a second, separate event on mount.

### F22. Rap completed fires on failure too, and callback(true) hides later-step failures
**Medium.** _Confirmed by the verifier._
executeENSRap sends 'Rap completed' unconditionally after the actions run. It sends it when the first action failed (callback(false, error)). It also sends it when later actions threw, because executeAction swallows those errors and the loop still ends with callback(true). A failed run therefore emits both 'Rap failed' and 'Rap completed'. Completed counts equal started counts, so a completion rate is always about 100% and the failure rate can't be read from it. 'Rap failed' also fires once per failing action, not once per rap.
- **Evidence:** `src/features/ens/raps/actions.ts:742`, `src/features/ens/raps/actions.ts:689`, `src/features/ens/raps/actions.ts:735`, `src/features/ens/raps/actions.ts:738`
- **Fix:** Send the end event with a status of success, failed or partial, or send it only when every action succeeded. Make executeAction return per-action errors so the final status is true. Send one failure event per rap, or add the step index to each failure.
- **Items:** `Rap completed`, `Rap failed`, `Rap started`
- **Verifier:** actions.ts:687-700 swallows errors after the first action, the loop at 730-735 ignores failures and calls callback(true), and line 742 sends Rap completed unconditionally.

### F23. predictions.place_order.failed can fire for an order that already matched and sent predictions.place_order
**Medium.** _Confirmed by the verifier._
executePolymarketBuyPosition and executePolymarketSellPosition start matched-order analytics inside the call. If the accepted response is already matched with hashes, predictions.place_order fires right then. The sheet then awaits waitForPositionSizeUpdate in the same try. If that throws, the catch sends predictions.place_order.failed for the same order. One order can therefore count as both placed and failed, and the later order_match.failed poll can add a third outcome. Place-order success rate and failure counts are inflated and no longer add up per order. The failure also cannot tell a rejected submission from a late position-size timeout.
- **Evidence:** `src/features/polymarket/utils/executePolymarketOrder.ts:121`, `src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:201`, `src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:209`, `src/features/polymarket/screens/polymarket-sell-position-sheet/PolymarketSellPositionSheet.tsx:120`, `src/features/polymarket/screens/polymarket-sell-position-sheet/PolymarketSellPositionSheet.tsx:127`
- **Fix:** Wrap only the order execution in the try that sends predictions.place_order.failed. Handle waitForPositionSizeUpdate separately: log it and show an alert, but send no failure event. You can also send a `stage` property so the two causes can be told apart.
- **Items:** `predictions.place_order.failed`, `predictions.place_order`, `predictions.order_match.failed`
- **Verifier:** PolymarketNewPositionSheet.tsx:201 awaits waitForPositionSizeUpdate inside the same try whose catch (209) sends place_order.failed. executePolymarketOrder.ts:121 starts the matched-order tracking inside execute.

### F24. swaps.quote_failed duplicates received_quote and its registry entry omits its real properties
**Medium.** _Confirmed by the verifier._
`swaps.received_quote` is already sent with the quote response, including a QuoteError, before `analyticsTrackQuoteFailed` runs. So the failure appears on a separate event (R004), and quote failures are counted in two places. The registry entry lists only the wrapper properties. The code sends `error_code`, `reason`, `inputAsset`, `outputAsset`, `inputAmount` and `outputAmount`, so none of what answers its question is documented. These properties are snake_case while the rest of the swaps events use camelCase.
- **Evidence:** `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:38`, `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:47`, `src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438`, `src/analytics/event.ts:779`
- **Fix:** Document the real properties in the registry. Longer term, put `status` and `failure_reason` on `swaps.received_quote` and reduce the payload to flat fields. Deprecate `swaps.quote_failed` with `replaced_by` and a dual-fire window. Don't remove it abruptly, because existing reports use it.
- **Items:** `swaps.quote_failed`, `swaps.received_quote`
- **Verifier:** swaps.received_quote fires for error quotes (useSwapInputsController.ts:438, before the check at 444). The registry entry for quote_failed lists only the wrapper properties.

### F25. perps.closed_position fires before the close is confirmed, so one close can emit both a success and a failure event
**Medium.** _Confirmed by the verifier._
perps.closed_position is tracked right after closePosition resolves, before waitForTradeByOrderId, navigation and the trade-details step. Those later steps sit in the same try block. If waitForTradeByOrderId or anything after the track call throws, the catch branch also sends perps.close_position.failed. A single close then counts as both closed and failed. That inflates the failure rate and the closed-position totals and PnL sums, and a funnel on the pair will not add up. Failure is also split into two events, which goes against the one-outcome-event rule (status on one event).
- **Evidence:** `src/features/perps/screens/ClosePositionBottomSheet.tsx:150`, `src/features/perps/screens/ClosePositionBottomSheet.tsx:163`, `src/features/perps/screens/ClosePositionBottomSheet.tsx:179`
- **Fix:** Only the closePosition call should decide which event fires. Wrap the post-close steps (waitForTradeByOrderId, navigation) in their own try/catch so their errors don't reach the failure track. Moving the firing point changes what past data means. Add a changelog line starting 'SEMANTICS:' with the date and name the dashboards that use these events. If you split or rename events, deprecate the old entry with replaced_by and dual-fire for one release.
- **Items:** `perps.closed_position`, `perps.close_position.failed`
- **Verifier:** ClosePositionBottomSheet.tsx:150 sends closed_position, then waitForTradeByOrderId (163) and goBack (168) run in the same try, whose catch at 179 sends close_position.failed.

### F26. Add-to-position cancel is tracked as perps.open_position.canceled, so it can't be told apart from opening a new position
**Medium.** _Confirmed by the verifier._
When createIsolatedMarginPosition returns nothing, the add-to-position sheet sends perps.open_position.canceled with market, side, leverage and perpsBalance. PerpsNavigatorFooter.tsx:237 sends the same event for the open-new-position flow. No property says which flow it came from, so the cancel count for opening a position mixes in add-to-position cancels. The add-to-position funnel (intent, then added or failed) has no matching cancelled outcome of its own.
- **Evidence:** `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214`, `src/features/perps/components/PerpsNavigatorFooter.tsx:237`
- **Fix:** Add a property such as `flow = open | add` to perps.open_position.canceled, which is an additive change. Alternatively, give the add-to-position sheet an outcome with `status = cancelled` on perps.added_to_position. Don't rename the existing event. If its meaning changes, go through deprecate + replaced_by with a dual-fire window, and add a SEMANTICS changelog line.
- **Items:** `perps.added_to_position`, `perps.open_position.canceled`
- **Verifier:** PerpsAddToPositionSheet.tsx:213-219 sends perpsOpenPositionCanceled with no flow property.

### F27. rnbw_airdrop.claim.failed can fire after a claim that succeeded, and cancelled signing counts as a failure
**Medium.** _Confirmed by the verifier._
The post-claim balance refresh at line 90 runs inside the same try block as the claim. If it throws, the catch at line 116 sends rnbw_airdrop.claim.failed even though ClaimAirdrop was accepted and polled to completion. The success event at line 91 never fires, so the claim is recorded as failed and the success rate is understated. Separately, prepareAirdropClaim sends the failed event at line 49 when signing throws. A user who cancels signing lands there too, so cancellation inflates the failure count. Rule 4 asks for `cancelled` as its own status that is not a failure.
- **Evidence:** `src/features/rnbw-rewards/utils/claimAirdrop.ts:90`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:116`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:49`
- **Fix:** Move the balance refresh after the success track, or wrap it in its own try/catch so it cannot send the failure event. Tell cancelled signing apart from failure, for example with a `status = cancelled` value or a `failure_reason`. Adding a value is additive, not breaking. Add a `SEMANTICS:` changelog line, because the firing moment of both events changes.
- **Items:** `rnbw_airdrop.claim.failed`, `rnbw_airdrop.claim`
- **Verifier:** The refresh part is the same as F15 (claimAirdrop.ts:90/116). The new part holds: signMessage at line 40 throws into the catch at 49 with no cancel distinction.

### F28. rnbw_rewards.claim does not say whether rewards went to the wallet or into staking
**Medium.** _Confirmed by the verifier._
submitRewardsClaim picks the endpoint from claimToDestination (default 'wallet'). The same rnbw_rewards.claim and rnbw_rewards.claim.failed events fire for a wallet claim and for a stake-rewards call. Neither payload includes claimToDestination or the endpoint. Claim counts and success rates mix two different actions and cannot be split. claimToDestination is already available in the function and is sent on rnbw_staking.stake.
- **Evidence:** `src/features/rnbw-rewards/utils/claimRewards.ts:123`, `src/features/rnbw-rewards/utils/claimRewards.ts:151`, `src/analytics/event.ts:1462`
- **Fix:** Add an optional claimToDestination ('wallet' | 'staking') property to both events and to the typed payloads. Adding a property is additive, not breaking.
- **Items:** `rnbw_rewards.claim`, `rnbw_rewards.claim.failed`
- **Verifier:** claimRewards.ts:98 picks the endpoint from claimToDestination, and neither the success call at 123 nor the failed call at 151 sends it.

### F29. Raw exception text is sent as the `error` property on two events
**Medium.** _Confirmed by the verifier._
The end-of-flow signing event sends `error?.message` straight from whatever the final step threw (signing, keychain, broadcast or RPC errors). wallet_initialization.failed sends `error.message` too. Exception messages are free text and can carry addresses, URLs or RPC payloads. They also have unbounded cardinality, so they can't be used for a breakdown. The design rules ask for failure reasons read from error enums or codes. The operation event also forwards the caller's free-form `metadata` object unchanged (line 56). I did not check the callers' metadata values.
- **Evidence:** `src/state/performance/performance.ts:74`, `src/state/wallets/initializeWallet.ts:191`, `src/state/performance/performance.ts:56`
- **Fix:** Replace the message with a bounded `error_reason` enum (error name or code, such as WalletAlreadyExistsError, user_rejected or network). Truncate or drop the raw text. Whitelist the keys allowed in `metadata`. Renaming or retyping an existing property is a breaking change. Add the new property next to the old one for a release, mark the old one deprecated, and move reports over before removing it.
- **Items:** `performance.time_to_sign`, `wallet_initialization.failed`
- **Verifier:** performance.ts:74 sends `error?.message`, line 56 forwards the caller's `metadata`, and initializeWallet.ts:191 sends `error.message`.

### F30. Error boundary events send the raw error object
**Medium.** _Confirmed by the verifier._
`onError` and `onReset` pass the caught `error` (typed `unknown`) and the full React `componentStack` as event properties. An error object can carry user-specific text such as addresses, API payloads or messages. The error object is serialised unpredictably by the PostHog client, often as `{}` or with a large stack. Sentry already receives the full error in the same boundary, so the analytics copy adds risk and little value.
- **Evidence:** `src/app/error-boundary/ErrorBoundary.tsx:14`, `src/app/error-boundary/ErrorBoundary.tsx:21`
- **Fix:** Send `error_name` (the error class name) and `eventId`, which links to Sentry, and drop the message and stack. Leave the full detail to Sentry.
- **Items:** `error_boundary.viewed`, `error_boundary.reset`
- **Verifier:** ErrorBoundary.tsx:14-25 sends the raw `error` (typed unknown) and the full componentStack.

### F31. `cashAmountEntered` fires on hold-to-add, before the wallet check, so it is an intent signal named like an input step
**Medium.** _Confirmed by the verifier._
`cashAmountEntered` is tracked inside `handleHoldToAdd`, once the user has held to confirm and before `checkWalletLink` runs. It is not tracked when the amount is typed. It fires again on every hold attempt, including ones that go on to a wallet-link sheet or a wallet-check failure. Analysts reading it as 'amount entered' will count confirmation attempts. The paired outcome is split across `cashWalletCheckFailed` and the buy-order action, which is not in this code, so I couldn't check it.
- **Evidence:** `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477`, `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:481`, `src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:495`
- **Fix:** State in the registry description that the event is the hold-to-add confirmation. Longer term, rename it through deprecate + `replaced_by` (for example `cash_deposit_started`) and put its failure on the outcome event with `status` and `failure_reason`.
- **Items:** `cash.amount_entered`, `cash.wallet_check_failed`
- **Verifier:** The registry and the cited lines put cashAmountEntered in handleHoldToAdd, before checkWalletLink.

### F32. Failed or errored signing is counted as a user rejection (`request.rejected`)
**Medium.** _Confirmed by the verifier._
When signing returns no result, or the wallet or keychain throws, the code calls `onCancel(response?.error)`. That sends `txRequestReject` with `requestType` and `isHardwareWallet`, and no reason or error. Hardware wallet failures and signing errors are therefore mixed with real user rejections. The rejection rate and the approve vs. reject funnel are skewed, and the send-transaction path has no failure event at all. The `txRequestApprove` call for transactions also sits after a try/catch that swallows errors, so it depends on `response?.result` being set.
- **Evidence:** `src/features/dapp-request/screens/SignTransactionSheet.tsx:295`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:362`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:406`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:523`
- **Fix:** Give the outcome a `status = approved | rejected | failed` and a `failure_reason`, or pass a `reason` to the reject event. Keep the existing event names (deprecate + `replaced_by` and dual-fire if they change).
- **Items:** `txRequestReject`, `txRequestApprove`
- **Verifier:** SignTransactionSheet.tsx:288-301: onCancel(error) always sends txRequestReject with no reason. Line 448 skips onCancel for hardware wallets, and line 523 calls onCancel when message signing returns no result.

### F33. importedSeedPhrase fires for every import type, and tappedImportButton fires before validation
**Medium.** _Confirmed by the verifier._
`importedSeedPhrase` is tracked at the end of `handleImportSuccess` for every successful import. The `isValidAddress`, ENS and Unstoppable checks only decide whether to show the backup prompt. So watch-only and address imports are counted as seed phrase imports. `tappedImportButton` fires before the `!isSecretValid || !seedPhrase` early return, so taps that never start an import are counted. The intent event has no matching outcome with a status or failure_reason. Failed imports (the `setBusy(false)` and Alert branches) leave no event.
- **Evidence:** `src/hooks/useImportingWallet.ts:337`, `src/hooks/useImportingWallet.ts:321`, `src/hooks/useImportingWallet.ts:132`, `src/hooks/useImportingWallet.ts:134`
- **Fix:** Add an `import_type` property (seed, private_key, address, ens, hardware) to the outcome event and a `status`/`failure_reason` for failed imports. Move the intent event after validation, or add a `valid` property. Both change what the existing events count. Record this with a `SEMANTICS:` changelog line, keep the old event name, and dual-fire for one release.
- **Items:** `importedSeedPhrase`, `tappedImportButton`
- **Verifier:** useImportingWallet.ts:337 sends importedSeedPhrase for every successful import. Line 132 sends tappedImportButton before the validity check at 134.

### F34. Amount-input events fire on every keystroke
**Medium.** _Confirmed by the verifier._
`onChangeNativeAmount` and `onChangeAssetAmount` run on each change of the text input and call `analytics.track` unconditionally. One send produces one event per digit typed. The rules say a text input should fire once, when it settles, and only if the value changed. These events add volume and no question anyone asks. The send outcome is already tracked by `sentTransaction`.
- **Evidence:** `src/features/transfer/screens/SendSheet.tsx:357`, `src/features/transfer/screens/SendSheet.tsx:378`
- **Fix:** Remove the calls, or debounce them to fire once after the input settles. If you remove them, deprecate both events with `deprecated_reason` and remove the calls in the same PR.
- **Items:** `changedNativeCurrencyInputSend`, `changedTokenInputSend`
- **Verifier:** SendSheet.tsx:357 and :378 send on every change, with no debounce.

### F35. WalletConnect failures are split across events, double-counted and use free-text reasons
**Medium.** _Confirmed by the verifier._
When namespaces are invalid, `rejectProposal` already tracks `wcNewSessionRejected`. The code then also tracks `wcRequestFailed` for the same failure, so one failure is counted twice. `wcRequestFailed` is used as a catch-all with a free-form `type` (`'session_request'`, `'read only wallet'`, a template literal for `invalid namespaces`, a template literal for `method not supported`). Its `reason` is a hard-coded sentence or a runtime `namespaces.error.message`. The result is unbounded values that split breakdowns. The `_failed` event shape also goes against the rule of one outcome event with `status` and `failure_reason`. `dappName` and `dappUrl` are sent on the other events with the same wallet context.
- **Evidence:** `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:175`, `src/features/wallet-connect/handlers/onSessionRequest.ts:84`, `src/features/wallet-connect/handlers/onSessionRequest.ts:119`, `src/features/wallet-connect/handlers/onSessionRequest.ts:147`, `src/features/wallet-connect/handlers/onSessionRequest.ts:214`
- **Fix:** Use a closed enum for `failure_reason` (invalid_params, read_only_wallet, session_not_found, unsupported_method, invalid_namespaces) and keep the message out of the event. Decide whether the proposal path emits `wcNewSessionRejected` or `wcRequestFailed`, not both. The change alters existing events: keep the old names, add the new properties, and dual-fire for one release.
- **Items:** `wcRequestFailed`, `wcNewSessionRejected`
- **Verifier:** onSessionProposal.ts:170-175: rejectProposal sends wcNewSessionRejected (62) and then wcRequestFailed is sent with the free-form `namespaces.error.message`.

### F36. ENS names and token names are sent as event properties
**Medium.** _Confirmed by the verifier._
viewedEnsProfile sends the user's own ENS name as `ens`. That is a human-readable identifier tied to the person and their wallet. addFavoriteToken sends the token address, name and symbol, and the name and symbol come from remote metadata. The ENS name is the real privacy concern. The token fields are high-cardinality and add little a breakdown needs.
- **Evidence:** `src/hooks/useOnAvatarPress.ts:111`, `src/resources/favorites.ts:178`
- **Fix:** Replace `ens` with a boolean such as `has_ens`, or drop it. Keep the token address and chainId and drop name and symbol. These changes alter existing events, so use dual-fire or a registry changelog entry rather than silently dropping the property.
- **Items:** `viewedEnsProfile`, `addFavoriteToken`
- **Verifier:** useOnAvatarPress.ts:113 sends the ENS name as `ens`. The registry shows addFavoriteToken sending the token name and symbol.

### F37. Claim failure event mixes user cancellations with real failures, and success/failure are two events
**Medium.** _Confirmed by the verifier._
The catch block computes `isUserCancelled` and logs it, but the analytics call never sends it. A user rejecting the signature therefore counts as `claimClaimableFailed`, which inflates the failure rate. The outcome is also split into a `Succeeded` event and a `Failed` event, so conversion rates need a union. It is money-adjacent, so the rules want one outcome event with `status = success | failed | cancelled` and a `failure_reason`.
- **Evidence:** `src/screens/Airdrops/utils.ts:140`, `src/screens/Airdrops/utils.ts:121`, `src/screens/Airdrops/utils.ts:150`
- **Fix:** Add `status` and `failure_reason` to the claim outcome, with `cancelled` when `isUserCancelled` is true. Treat it as a modification of an existing event: keep the old events, deprecate them with `replaced_by`, and dual-fire for one release.
- **Items:** `claimClaimableSucceeded`, `claimClaimableFailed`
- **Verifier:** Airdrops/utils.ts:140-160 computes isUserCancelled but only logs it. The failure event always uses failureStep 'claim'.

### F38. Claim failure is a separate event from claim success (rule 4), with inconsistent `claimableType`
**Medium.** _Confirmed by the verifier._
The claim flow sends `claimClaimableSucceeded` and `claimClaimableFailed` as two events, so a claim success rate needs a union of both. `claimableType` also has two sources in the transaction flow. Failures hard-code 'transaction', but success sends `claimable.actionType`. A breakdown by `claimableType` therefore splits success and failure into different buckets, and the failure rate per type is wrong. Failure reasons are the `failureStep` and `errorMessage` properties, which is the right data but on the wrong event.
- **Evidence:** `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:469`, `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:417`, `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:136`
- **Fix:** Add a `claim_claimable_completed` outcome event with `status = success | failed`, `failure_step` and `error_message`, and use one `claimableType` source on both paths. Don't rename in place. Deprecate the two old events with `replaced_by`, and dual-fire for one release because this is a money-adjacent event.
- **Items:** `claim_claimable.failed`, `claim_claimable.succeeded`
- **Verifier:** TransactionClaimableContext.tsx: failures hard-code `claimableType: 'transaction'` (418, 441), while success sends `claimable.actionType` (470).

### F39. Token launcher events send free text typed by the user
**Medium.** _Confirmed by the verifier._
getAnalyticsParams() returns name, symbol, description, logoUrl and the link URLs the user typed. Both the success and the failure event spread these into the event properties. The failure event also sends the raw error.message. The rules say never to send raw user text as event properties. On the failure path the token was never published, so this text is private at that point. Free text also makes the property values unbounded.
- **Evidence:** `src/screens/token-launcher/state/tokenLauncherStore.ts:247-256`, `src/screens/token-launcher/state/tokenLauncherStore.ts:379`, `src/screens/token-launcher/state/tokenLauncherStore.ts:438`
- **Fix:** Send has_description, has_links, name_char_count and link_count in place of the text. Send an error code or enum as failure_reason in place of error.message. The creation event carries both success and failure. Per rule 4 it should be one outcome event with status and failure_reason. Changing the event shape is a breaking change, so go through deprecate + replaced_by with a dual-fire window.
- **Items:** `tokenLauncherTokenCreated`, `tokenLauncherCreationFailed`
- **Verifier:** tokenLauncherStore.ts:379-381 and :438-440 spread analyticsParams and send `error.message`. The registry describes the payload as the whole form state, but I did not open getAnalyticsParams at :247-256.

### F40. Wallet profile submit and cancel events record taps only and cannot be turned into a rate
**Medium.** _Confirmed by the verifier._
Submit fires at the start of handleSubmit, before onCloseModal saves the profile and before the keychain callback runs, so a failed save still counts. Cancel fires after goBack(). Nothing fires when the modal opens, so there is no denominator for abandonment. Neither call passes properties, so actionType (Create or edit) and isFromSettings are not recorded, even though both change the flow.
- **Evidence:** `src/components/expanded-state/WalletProfileState.js:54`, `src/components/expanded-state/WalletProfileState.js:61`, `src/components/expanded-state/WalletProfileState.js:68`
- **Fix:** Add `action_type` and `entry_point` (settings or wallet switcher) as properties. Add an open/start event, or move the outcome event to the saved point with `status`. Do this as new events or extra properties, not by changing the old firing moment in place. If the moment does change, add a `SEMANTICS:` changelog line.
- **Items:** `Tapped "Submit" on Wallet Profile modal`, `Tapped "Cancel" on Wallet Profile modal`
- **Verifier:** WalletProfileState.js:61 sends Submit before onCloseModal and the keychain callback, line 54 sends Cancel after goBack, and neither call sends properties.

### F41. Registry property lists don't match what the code sends
**Medium.** _Confirmed by the verifier._
initializeWallet passes `walletStatus` to finishMeasuring, and the event property type declares it. The registry has no walletStatus property. The registry's own question ('new, imported and existing wallets') can only be answered with that property. performance.report is sent with `reportName`, `segments` and `data` (the per-segment timings and extra params, such as the initial route). The registry lists none of them, so the per-segment breakdown that the question depends on is undocumented. The registry instead lists walletAddressHash and walletType as event properties of each event. Those are not passed by the calls. They come from the analytics default metadata, so they are context properties, not part of the event payload.
- **Evidence:** `src/state/wallets/initializeWallet.ts:173`, `src/analytics/event.ts:948`, `src/analytics/event.ts:953`, `src/performance/tracking/index.ts:184`
- **Fix:** Add walletStatus (new | imported | existing, from getWalletStatus) to 'Performance Wallet Initialize Time'. Add reportName, segments and data to 'performance.report'. Note that the `segments` keys are the PerformanceReportSegments values and `data` is free-form. Keep walletAddressHash and walletType documented as default metadata. Additive change, not breaking.
- **Items:** `Performance Wallet Initialize Time`, `performance.report`
- **Verifier:** initializeWallet.ts:173-175 passes walletStatus, and trackMetric spreads it into the event (performance/tracking/index.ts:79-86). The registry doesn't list it.

### F42. Rejected new WalletConnect session also fires for user rejections and double-counts with wc.failed_request
**Medium.** _Confirmed by the verifier._
The plan says the event fires only for invalid namespaces and not when the user dismisses the sheet. rejectProposal is also called when the person rejects (approved is false, reason USER_REJECTED), so the event covers both app refusals and user rejections. It carries no reason, so the two cannot be told apart. On the invalid-namespaces path the same refusal sends this event inside rejectProposal and then sends wc.failed_request, so one refusal is counted in two events. Rejection counts and failure counts are inflated and cannot be separated.
- **Evidence:** `src/features/wallet-connect/handlers/onSessionProposal.ts:205`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:170`, `src/features/wallet-connect/handlers/onSessionProposal.ts:175`
- **Fix:** Pass the rejection reason (USER_REJECTED vs INVALID_SESSION_SETTLE_REQUEST) to rejectProposal and send it as a property on the rejected event. Drop the extra wc.failed_request on the invalid-namespaces path, or document that the two events overlap. Update the plan to say user rejections are included.
- **Items:** `Rejected new WalletConnect session`, `wc.failed_request`
- **Verifier:** onSessionProposal.ts:205 calls rejectProposal with USER_REJECTED, so the rejected event (62) also counts user rejections and carries no reason. On invalid namespaces it is double-counted with wcRequestFailed (170-175).

### F43. performance.time_to_sign and performance.time_to_sign.operation are never sent when an async step rejects
**Medium.** _Confirmed by the verifier._
The plan says these events fire when a timed step returns, resolves or throws. In executeFn, a synchronous throw calls recordPerformance({ error }). An async step that rejects does not. The promise chain only calls recordPerformance in .then (success), and .catch just rethrows. A failed async step, such as a failed sign or broadcast, is therefore never recorded. If it was the last step, no time_to_sign is sent, and the elapsed-time counters are not reset. Timing data is biased toward successful flows. Counters can leak into the next flow unless startTimeToSignTracking resets them. The `error` property is also only ever set on the synchronous-throw path.
- **Evidence:** `src/state/performance/performance.ts:118`, `src/state/performance/performance.ts:121`, `src/state/performance/performance.ts:124`, `src/state/performance/performance.ts:133`
- **Fix:** Call recordPerformance({ error: ensureError(e) }) inside the promise .catch before rethrowing. Replace the no-op .catch(e => { throw e; }).
- **Items:** `performance.time_to_sign`, `performance.time_to_sign.operation`
- **Verifier:** performance.ts:118-126: the promise `.catch` only rethrows and never calls recordPerformance, so a rejected async step sends no events and doesn't reset the counters.

### F44. swaps.selected_asset fires on every setAsset call, even with no change, and twice on a flip
**Medium.** _Confirmed by the verifier._
analytics.track in setStoreAssets sits outside the didSelectedAssetChange check, so it fires when the person re-picks the same token and when asset is null. flipWithTracking calls setAsset and then sends swaps.flipped_assets, so one flip press sends both selected_asset and flipped_assets. Selected-asset counts are inflated and mix real selections with flips.
- **Evidence:** `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1004`, `src/__swaps__/screens/Swap/components/FlipButton.tsx:50`, `src/__swaps__/screens/Swap/components/FlipButton.tsx:97`
- **Fix:** Move the track inside `if (didSelectedAssetChange)`. Skip it when didFlipAssets, or add a flag so flips don't send selected_asset.
- **Items:** `swaps.selected_asset`, `swaps.flipped_assets`
- **Verifier:** swap-provider.tsx:1004-1027: the track call sits outside `if (didSelectedAssetChange)`.

### F45. token_list on the Send screen counts 'no_price' from the 24h change, not from the price
**Medium.** _Confirmed by the verifier._
The Send sheet increments no_price when `!asset.price?.relative_change_24h` is true. That field is the 24h percentage change. A token with a valid price and a 0% or missing change is counted as having no price. The registry describes no_price as a count of tokens with no price. So the send series is not comparable with the other screens. The same effect re-runs whenever sortedAssets changes, so one visit can send several snapshots and inflate counts per user.
- **Evidence:** `src/features/transfer/screens/SendSheet.tsx:668`, `src/features/transfer/screens/SendSheet.tsx:663`
- **Fix:** Test the price itself (for example `typeof asset.price?.value !== 'number'`) and align it with the wallet and discover senders. Fire once per screen visit with a ref guard. This changes what no_price means for past data. Add a changelog line starting `SEMANTICS:` and note the change in the registry description.
- **Items:** `token_list`
- **Verifier:** SendSheet.tsx:668 counts `!asset.price?.relative_change_24h` as no_price, and the effect at 663-671 re-runs whenever sortedAssets changes.

### F46. swaps.submitted amounts are raw smallest-unit values, and errorMessage is always null
**Medium.** _Confirmed by the verifier._
inputAssetAmount and outputAssetAmount are Number(quote.sellAmount) and Number(quote.buyAmount), the raw amounts in each token's smallest unit. Summing or averaging them across tokens with different decimals gives meaningless totals. They can also exceed safe-integer precision for 18-decimal tokens. errorMessage is typed `undefined` for this event and sent as `errorMessage ?? null`, so it is always null and carries no information. The same params builder is shared with swaps.failed.
- **Evidence:** `src/__swaps__/utils/trackSwapEvent.ts:51`, `src/__swaps__/utils/trackSwapEvent.ts:62`, `src/__swaps__/utils/trackSwapEvent.ts:49`, `src/__swaps__/utils/trackSwapEvent.ts:15`
- **Fix:** Add new properties for decimal-adjusted amounts (e.g. `input_amount_decimal`). Don't change the type or unit of the existing names, because analytics tools can't retype history. Stop sending errorMessage on swaps.submitted, or document it as always null. Rely on tradeAmountUSD for value reports.
- **Items:** `swaps.submitted`
- **Verifier:** trackSwapEvent.ts:51/62 send `Number(quote.sellAmount/buyAmount)` as raw base-unit amounts. Line 49 sets errorMessage to null on submitted (typed undefined at :15).

### F47. cash.buy_completed is silently skipped when a completed order lacks amounts or timestamps
**Medium.** _Confirmed by the verifier._
applyTerminalOrder only tracks when the order has cryptoAmount, fiatAmount, createdTime and completedTime. The order still goes to the success state and the purchase is still queued as a pending transaction. A completed order missing any field sends nothing. That undercounts this money outcome, with no failure event to show the gap. A buy_submitted vs buy_completed funnel would show a drop-off that isn't real.
- **Evidence:** `src/features/cash/stores/cashBuyOrderStore.ts:122`, `src/features/cash/stores/cashBuyOrderStore.ts:143`
- **Fix:** Fire cash.buy_completed whenever status is Completed. Make the amount and timeToUsdcMs properties optional, or add a `status` property, rather than dropping the event. Because this changes the event's firing moment, add a `SEMANTICS:` changelog line with the date. Ideally this money outcome would also be sent from the server.
- **Items:** `cash.buy_completed`
- **Verifier:** cashBuyOrderStore.ts:122 skips the completed event when any field is missing, while line 143 still sets success and has no else branch.

### F48. wc.failed_request is a failure-only event with no outcome pair, and its failures lack dapp context
**Medium.** _Confirmed by the verifier._
The call sites send only type and reason. The question in the registry asks 'from which dapps', but dappName and dappUrl are not sent on any failure, although the data is available in the proposer metadata (onSessionProposal.ts:63-64) and in the session peer metadata. No call is made when the try/catch around approveSession fails, so that failure is invisible. The type union includes 'session_proposal' but nothing sends it. For invalid namespaces, reason is the free-form validator message (onSessionProposal.ts:175), so breakdowns by reason will split. The event has a start and a success (Approved new WalletConnect session, Showing signing request), so it is closer to a rule 4 failure event that is split from its outcomes. That is acceptable here because no single outcome event exists.
- **Evidence:** `src/features/wallet-connect/handlers/onSessionProposal.ts:175`, `src/features/wallet-connect/handlers/onSessionRequest.ts:214`, `src/analytics/event.ts:661`
- **Fix:** Add dappName and dappUrl, which are in scope, to the failure calls. Send a fixed enum for invalid-namespace reasons instead of the validator message. Track the approveSession catch branch. Do this additively, with no rename, so existing data stays comparable.
- **Items:** `wc.failed_request`
- **Verifier:** The approveSession catch (onSessionProposal.ts:184-203) only alerts and logs. The invalid-namespaces reason (175) is free-form. The registry and type have no dapp fields on failure calls.

### F49. Changed Global Notification Settings can never fire: the global topic list is empty
**Medium.** _Confirmed by the verifier._
`GlobalNotificationTopic` is `{}`, so `GlobalNotificationTopicType` has no values and the Settings screen has no global topic to toggle. The only call site is `toggleGlobalNotificationTopic`, after a successful publish, so the event is dead tracking. The `topic` property also has no possible values. Nothing is known about arriving data, because there is no snapshot.
- **Evidence:** `src/notifications/settings/constants.ts:11`, `src/notifications/settings/settings.ts:104`, `src/screens/SettingsSheet/components/NotificationsSection.tsx:262`, `src/notifications/analytics.ts:17`
- **Fix:** If global topics are gone for good, mark the event `deprecated` with a `deprecated_reason` (no replacement) and remove the call in the same PR, per the removal rule. Keep the registry entry. If topics are coming back, keep the event and the registry draft, and note that it is dormant. Per-wallet topic toggles should go through a separate decision.
- **Items:** `Changed Global Notification Settings`
- **Verifier:** constants.ts:11 defines `GlobalNotificationTopic = {}`, so there are no global topics.

### F50. learn_article.opened fires when the screen closes, and can fire more than once
**Medium.** _Confirmed by the verifier._
The event is sent from the cleanup function of a useEffect, so it fires on unmount, not on open. The effect's dependencies are [category, displayType, key, routeName, url]. If any of them change while the screen is open, the cleanup runs and sends an extra event with a shortened duration. Counts of 'opened' are therefore really counts of closes plus any param changes, and an article that is opened and never closed (app killed) isn't counted.
- **Evidence:** `src/screens/LearnWebViewScreen.tsx:33`, `src/screens/LearnWebViewScreen.tsx:35`, `src/screens/LearnWebViewScreen.tsx:45`
- **Fix:** Fire an opened event on mount (empty-deps effect) and send duration on a separate close or read event, or keep one event but read the params through refs with an empty dependency list so it fires once on unmount. Changing the firing moment is breaking for existing reports: add a new event and mark this one deprecated with replaced_by, dual-fire for one release, and add a SEMANTICS: changelog line.
- **Items:** `learn_article.opened`
- **Verifier:** LearnWebViewScreen.tsx:33-45 sends the event from a cleanup with dependencies [category, displayType, key, routeName, url].

### F51. `no_price` in the Discover token_list event counts the opposite of its name
**Medium.** _Confirmed by the verifier._
The loop increments `no_price` when `!isNaN(asset.price?.value)`. That condition is true when the asset HAS a price. A missing price gives `undefined`, `isNaN(undefined)` is true, and the counter is skipped. The swap-side hook counts `no_icon` correctly. In the Discover event, `no_price` holds the number of priced tokens, so any report on tokens without prices is inverted.
- **Evidence:** `src/components/Discover/DiscoverSearch.tsx:115`
- **Fix:** Change the check to `if (isNaN(asset.price?.value)) params.no_price += 1`. Because this changes what a property means, add a SEMANTICS changelog line with the date. Add a new property name if past `no_price` values must stay comparable, since an inverted history cannot be corrected in place.
- **Items:** `token_list`
- **Verifier:** DiscoverSearch.tsx:115 uses `!isNaN(asset.price?.value)`, which is true when the price is a number, so the count is inverted.

### F52. The Discover `Search Query` event fires on clearing the input, not when a search settles
**Medium.** _Confirmed by the verifier._
The event is sent from `handleClearInput` only when the user clears the input with more than one character. Searches that end by navigating away, selecting a result or dismissing the keyboard are never counted. The event also uses a different name style ('Search Query') from the rest of the plan. Taken as a count of searches, it undercounts and is biased.
- **Evidence:** `src/components/Discover/DiscoverSearchInput.tsx:133`
- **Fix:** Fire once when the query settles (debounced, and only if it changed) with `char_count` and `result_count`. Treat this as a firing-moment change, so add a SEMANTICS changelog line. Add a new event in the object_action convention and deprecate the old name with `replaced_by`.
- **Items:** `Search Query`
- **Verifier:** The registry entry and DiscoverSearchInput.tsx:133-134 show it fires only from handleClearInput when more than 1 character is typed.

### F53. Trending dapp click is never tracked for the first card (`index &&` guard on a 0-based index)
**Medium.** _Confirmed by the verifier._
The call is wrapped in `index && analytics.track(...)`. `index` comes from `.map((dApp, index) => ...)`, so it starts at 0. The first trending dapp has `index = 0`, which is falsy, so its clicks are dropped. The top-ranked and probably most-clicked slot is missing from counts and from any breakdown by `index`. The guard was probably meant to skip a missing index, since the prop is optional.
- **Evidence:** `src/features/dapp-browser/components/Homepage.tsx:333`, `src/features/dapp-browser/components/Homepage.tsx:159`, `src/features/dapp-browser/components/Homepage.tsx:160`
- **Fix:** Guard with `index !== undefined`. This changes the event's firing population, so add a `SEMANTICS:` changelog line with the date. Position 0 will appear in charts from that release on.
- **Items:** `browserTrendingDappClicked`
- **Verifier:** Homepage.tsx:333 uses an `index &&` guard, and line 160 passes the 0-based map index.

### F54. Token list event fires on every asset change
**Medium.** _Confirmed by the verifier._
The effect depends on `sortedAssets`. It re-runs, and sends tokenList, whenever the asset array changes identity. That happens on refreshes, price updates and sorting, not only when the screen is shown. Counts of this event follow data churn, and the no_icon and no_price totals are repeated.
- **Evidence:** `src/hooks/useWalletSectionsData.ts:117`, `src/hooks/useWalletSectionsData.ts:125`
- **Fix:** Fire once per screen focus, or once per wallet and address after loading finishes. Use a ref guard and keep `sortedAssets` out of the dependencies. This changes the firing moment, so add a `SEMANTICS:` changelog line.
- **Items:** `tokenList`
- **Verifier:** useWalletSectionsData.ts:117-126 depends on sortedAssets with no once-per-visit guard.

### F55. addWalletFlowStarted is mislabeled and inconsistent between entry points
**Medium.** _Confirmed by the verifier._
Restoring from cloud backup sends `type: 'seed'`, the same value as restoring from a seed phrase, so the two paths cannot be told apart. The hardware-wallet path hard-codes `isFirstWallet: false` while the sheet receives `isFirstWallet` as a route param and passes it on to the navigator. Create-new fires both `addWalletFlowStarted` and `tappedCreateNewWallet`, and restore and watch fire both `addWalletFlowStarted` and a `tapped*` event, so each action is counted twice. The settings backup screen fires `addWalletFlowStarted` with `type: 'new'` and `isFirstWallet: false` but sends no `entry_point`, so it cannot be separated from the AddWalletSheet path.
- **Evidence:** `src/screens/AddWalletSheet.tsx:82`, `src/screens/AddWalletSheet.tsx:84`, `src/screens/AddWalletSheet.tsx:99`, `src/screens/AddWalletSheet.tsx:107`, `src/screens/AddWalletSheet.tsx:45`, `src/screens/AddWalletSheet.tsx:49`, `src/screens/SettingsSheet/components/Backups/ViewWalletBackup.tsx:140`
- **Fix:** Give cloud restore its own `type` value such as `cloud`. Pass `isFirstWallet` through on the hardware path. Add an `entry_point` property on `addWalletFlowStarted` and drop the redundant `tapped*` events through deprecate + `replaced_by`. Changing existing values splits breakdowns, so add the new value and note it as a SEMANTICS change in the changelog.
- **Items:** `addWalletFlowStarted`
- **Verifier:** AddWalletSheet.tsx:82-85 sends 'seed' for the cloud restore, 98-101 hard-codes isFirstWallet false while passing the real value at 107, and 45-49, 58-59 and 70-71 send two events per tap.

### F56. wcRequestFailed fires on mount of an informational redirect sheet, not on a failure
**Medium.** _Confirmed by the verifier._
The 'no WalletConnect needed' sheet fires `wcRequestFailed` in a mount effect. No request was made. The user simply tried to use WalletConnect inside the dapp browser, which is a UX redirect. Counting it as a failure inflates WalletConnect failure counts for any other `type` or `reason` that uses the same event. `reason` is also a long free-text sentence rather than an enum.
- **Evidence:** `src/screens/NoNeedWCSheet.tsx:35`, `src/screens/NoNeedWCSheet.tsx:36`, `src/screens/NoNeedWCSheet.tsx:38`
- **Fix:** Move this to a dedicated event or to `error_occurred` (client-side refusal, no request formed), and use an enum `failure_reason` such as `wc_in_dapp_browser`. Treat it as a modification: deprecate the old usage with a dual-fire window.
- **Items:** `wcRequestFailed`
- **Verifier:** The registry entry shows NoNeedWCSheet:36 sends wcRequestFailed on mount, with a free-text reason.

### F57. Token launcher image-upload failure sends `url` and `isModerated`, which are always undefined
**Medium.** _Confirmed by the verifier._
`url` and `isModerated` are only assigned after the upload has succeeded. Every throw that reaches the catch happens before that, so both are undefined when the event is sent. The properties carry nothing, and moderation-related failure analysis isn't possible. The raw `error.message` is also sent as free text.
- **Evidence:** `src/screens/token-launcher/hooks/useUploadToCloudinary.ts:21`, `src/screens/token-launcher/hooks/useUploadToCloudinary.ts:29`
- **Fix:** Drop `url` and `isModerated` from the failure event, or read them from the response before throwing. Replace the free-text message with a bounded `failure_reason` enum.
- **Items:** `token_launcher.image_upload_failed`
- **Verifier:** useUploadToCloudinary.ts:14-33: url and isModerated are only assigned after a successful upload, so they are always undefined in the catch. error.message is sent as free text.

### F58. Raw error messages are sent as event properties on token-launcher failures
**Medium.** _Confirmed by the verifier._
`error` is set to `error.message` from an arbitrary caught exception. The exception comes from the wallet and private-key loading path, in a catch block that wraps `loadPrivateKey` and token creation. The text is unbounded, so it can't be broken down, and it can leak sensitive internals. The failure reasons are also not a closed set.
- **Evidence:** `src/screens/token-launcher/components/TokenLauncherFooter.tsx:120`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:79`
- **Fix:** Send a `failure_reason` enum such as `hardware_wallet_unsupported`, `private_key_null`, `not_authenticated` or `exception`. Keep the message in the logger only.
- **Items:** `token_launcher.wallet_load_failed`
- **Verifier:** TokenLauncherFooter.tsx:120 sends `error.message` from a catch that also wraps createToken (100).

### F59. `rewards.viewed_sheet` fires on every focus, not once per open
**Medium.** _Confirmed by the verifier._
The call is inside `useFocusEffect`, so it runs again every time the sheet regains focus. That happens after returning from the explainer sheet or the profile sheet opened from the rewards cards. Sheet views are over-counted.
- **Evidence:** `src/screens/rewards/RewardsSheet.tsx:43`, `src/screens/rewards/RewardsSheet.tsx:45`, `src/screens/rewards/components/RewardsClaimed.tsx:88`
- **Fix:** Fire once per mount with a `useEffect` and an empty dependency list, or guard it with a ref. Record the change with a `SEMANTICS:` changelog line, because past counts were inflated.
- **Items:** `rewards.viewed_sheet`
- **Verifier:** RewardsSheet.tsx:43-47 tracks inside useFocusEffect.

### F60. token_details.nft sends a nested `token` object that the registry does not document, including the NFT name and address
**Medium.** _Confirmed by the verifier._
The call passes `token: {isPoap, isParty, isENS, address, chainId, name, image_url}` alongside `available_data`, but the registry entry lists only available_data, eventSentAfterMs and the wallet context properties. The nested object makes breakdowns awkward. It also carries `name`, which for ENS and POAP assets can be user-chosen text. The 5s timer fires on mount and again via cleanup on early unmount. The call ignores the `cancelled` flag, so both cases look the same except for eventSentAfterMs.
- **Evidence:** `src/components/expanded-state/UniqueTokenExpandedState.tsx:389`, `src/components/expanded-state/UniqueTokenExpandedState.tsx:391`, `src/components/expanded-state/UniqueTokenExpandedState.tsx:385`
- **Fix:** Add `token` to the registry entry, or flatten it into scalar properties (`is_poap`, `is_ens`, `chain_id`). Drop `name` and `image_url` unless there is a stated question for them. Add a flag for the early-unmount case. Dual-fire any renamed property for one release.
- **Items:** `token_details.nft`
- **Verifier:** UniqueTokenExpandedState.tsx:389-393 sends a nested `token` object with name and image_url. useTimeout.ts:44-45 calls the callback on early unmount with `cancelled`, which is ignored.

### F61. Completion event has no failure path, and the documented SDK-callback firing is not found in the code
**Medium.** _Confirmed by the verifier._
The only call sites for `f2cProviderFlowCompleted` are the two branches in the deeplink handler. The comment there says providers with SDK callbacks (e.g. Ratio) fire the same event from the SDK, but no such call was found in `src`. The type allows `fiat_amount`, `fiat_currency` and other properties that no call site sends. For non-`ramp` providers `success` is omitted (unknown), and it is never `false`. So the event counts returns to the app, not successful purchases. Deeplinks missing `provider` or `sessionId` are only logged, with no event. `provider` is cast to `FiatProviderName` without validation, so a malformed link can send an unknown value. Abandonment can only be measured as `flow_opened` with no matching `flow_completed`.
- **Evidence:** `src/handlers/deeplinks.ts:151`, `src/handlers/deeplinks.ts:160`, `src/handlers/deeplinks.ts:172`, `src/handlers/deeplinks.ts:178`, `src/analytics/event.ts:488`
- **Fix:** Report completion rates as a per-provider return rate, not as success. Check whether the Ratio SDK callback call exists elsewhere. If it doesn't, either add it or remove the unused optional properties from the type. Add this as an extension of the existing event (no new event). Validate `provider` against the enum before tracking. Describe `success` as 'true only for ramp' in the registry.
- **Items:** `f2c.provider.flow_completed`
- **Verifier:** A grep finds f2cProviderFlowCompleted only at deeplinks.ts:172/178, so there is no SDK-callback call. `success` is never false, there is no event when params are missing (151-157), and provider is cast without validation.

### F62. BackupSheet shown is unreachable: BackupSheetSection is never rendered
**Medium.** _Confirmed by the verifier._
The only call sits in the useEffect of BackupSheetSection. Nothing in src imports or renders that component. A search for the name finds only its definition and two unrelated log strings. The event can never fire, so any report on how often people see the backup sheet reads zero. The registry profile lists the call as live.
- **Evidence:** `src/features/backup/components/BackupSheetSection.tsx:56`, `src/analytics/event.ts:276`
- **Fix:** If the sheet is still shown, send the event from the component that actually renders it (BackupSheet or BackupCloudStep), or render BackupSheetSection there. If the component is dead code, remove it and deprecate the event.
- **Items:** `BackupSheet shown`
- **Verifier:** A grep finds BackupSheetSection only in its own file, so nothing renders it.

### F63. Updated Gas Price also fires when a screen picks the gas speed automatically
**Medium.** _Confirmed by the verifier._
The event is sent from the shared Redux action gasUpdateGasFeeOption, so every caller sends it, not only a person choosing a speed. SpeedUpAndCancelSheet calls updateGasFeeOption(URGENT) from an effect once gas params load, with the comment 'Always default to urgent'. Every speed-up or cancel sheet open therefore sends the event with no tap. GasSpeedButton also delays one call by 500 ms. The count of gas changes is inflated, and 'urgent' is over-represented. The event name is swaps-specific, but the action runs on every screen with gas.
- **Evidence:** `src/features/gas/redux/gas.ts:488`, `src/screens/SpeedUpAndCancelSheet.tsx:359`
- **Fix:** Send the event from the places where the person picks a speed (GasSpeedButton and FeesPanelTabs handlers), or pass a source argument ('user' or 'default') and only track the user ones. Rename the event so it isn't swaps-specific.
- **Items:** `Updated Gas Price`
- **Verifier:** gas.ts:488 tracks inside the shared action, and SpeedUpAndCancelSheet.tsx:355-361 calls updateGasFeeOption(URGENT) from an effect with no tap.

### F64. request.sheet.show can fire several times for one sheet
**Medium.** _Confirmed by the verifier._
The tracking call is inside a useEffect whose dependencies include transactionDetails?.payload?.params (an array reference), fetchMethodName (which changes with chainId), startPollingGasFees and stopPollingGasFees. Whenever any of them changes identity, the effect re-runs and sends the event again. The runAfterInteractions callback is also never cancelled in the cleanup. The count of sheets shown is then inflated, and so is the baseline for the approve/reject rate.
- **Evidence:** `src/features/dapp-request/hooks/useTransactionSetup.ts:55`, `src/features/dapp-request/hooks/useTransactionSetup.ts:64`
- **Fix:** Send the event from a separate effect that fires once per sheet, for example guarded by a ref or with only the stable values it needs as dependencies. Keep the gas polling in the existing effect.
- **Items:** `request.sheet.show`
- **Verifier:** The useTransactionSetup.ts:46-64 effect dependencies include params, source and callbacks, and runAfterInteractions is not cancelled.

### F65. Completed ENS registration fires when the register tx is broadcast, not when the registration is done
**Medium.** _Confirmed by the verifier._
The event is sent right after executeRegisterWithConfig returns the broadcast transaction and the hash is saved. The transaction hasn't been mined yet, so a dropped, reverted or replaced transaction is still counted as a completed registration. The name says outcome, but the code only shows submission.
- **Evidence:** `src/features/ens/raps/actions.ts:405`, `src/features/ens/raps/actions.ts:422`
- **Fix:** Send the completion event from the place that sees the register transaction confirmed (the transaction watcher), with a status. Or rename it and document it as 'register submitted'.
- **Items:** `Completed ENS registration`
- **Verifier:** actions.ts:404-424 tracks right after executeRegisterWithConfig returns the broadcast tx, before confirmation.

### F66. claim_claimable.succeeded for airdrops is sent on submission, before on-chain confirmation
**Medium.** _Confirmed by the verifier._
The rainbowCoin airdrop flow sends succeeded right after provider.sendTransaction returns, before waitForTransaction. A transaction that later reverts or is dropped is still counted as a successful claim. The sponsored and transaction flows send it later, so the three claimableType values do not mean the same thing.
- **Evidence:** `src/screens/Airdrops/utils.ts:121`, `src/screens/Airdrops/utils.ts:133`
- **Fix:** Add a status or stage property (submitted vs confirmed), or send the success event from the waitForTransaction result and add a failure when the receipt shows a revert.
- **Items:** `claim_claimable.succeeded`
- **Verifier:** Airdrops/utils.ts:121 sends succeeded before waitForTransaction at 133.

### F67. Viewed profile can fire several times for one sheet open
**Medium.** _Confirmed by the verifier._
The effect depends on params, ensName, profileAddress and accountAddress. ensName resolves after the sheet opens, and a new params object also re-runs the effect. The sheet can therefore send two or more events per open, with the first carrying an empty or unresolved name. The profile view count is inflated and the name property is unreliable.
- **Evidence:** `src/screens/ProfileSheet.tsx:68`, `src/screens/ProfileSheet.tsx:74`
- **Fix:** Send once per open, for example guarded by a ref keyed on profileAddress. Or wait for the ENS name to resolve before sending. Drop params from the dependency list.
- **Items:** `Viewed profile`
- **Verifier:** ProfileSheet.tsx:66-74 has dependencies [params, ensName, profileAddress, accountAddress] and no guard.

### F68. token_list from Discover also fires when the timeout is cancelled
**Medium.** _Confirmed by the verifier._
useTimeoutEffect runs the callback with {cancelled: true} from its cleanup when the effect re-runs or unmounts before the 3 seconds are up. This happens when `enabled` (not loading) changes or when the person leaves Discover. The Discover callback ignores that argument. The snapshot therefore fires at once on an early leave or a loading flip, and can fire again later. It is meant to fire once, 3 seconds after the list is stable.
- **Evidence:** `src/hooks/useTimeout.ts:44`, `src/components/Discover/DiscoverSearch.tsx:102`, `src/components/Discover/DiscoverSearch.tsx:117`
- **Fix:** Take the argument in the callback and return early when `cancelled` is true, e.g. `({ cancelled }) => { if (cancelled) return; ... }`.
- **Items:** `token_list`
- **Verifier:** useTimeout.ts:40-46 calls the callback with `cancelled: true` on cleanup, and DiscoverSearch.tsx:103 ignores the argument.

### F69. direct_sdk flags are a script artefact: `analytics` is the tracking wrapper
**Low.** _Confirmed by the verifier._
All 30 flagged lines call `analytics.track` or `analytics.identify` on the `Analytics` class exported from `@/analytics`. That class is the single adapter in the repository: it is the only place that imports `posthog-react-native` and calls `client.capture` and `client.identify`. It also adds the wallet hash, device metadata and AppsFlyer id, and queues events until init finishes. The scanner appears to match the method names `track` and `identify` without noticing that `analytics` is the wrapper. These are correct wrapper calls, not SDK bypasses. The one wrapper-bypass is inside the adapter itself, at `src/analytics/index.ts:169`. It sends the literal 'Application Updated' through `client.capture`, which is harmless but not typed through `event.ts`.
- **Evidence:** `src/analytics/index.ts:3`, `src/analytics/index.ts:102`, `src/analytics/index.ts:107`, `src/analytics/index.ts:244`
- **Fix:** Teach the scanner that `analytics` from `@/analytics` is the wrapper and exclude src/analytics/index.ts from the direct_sdk check. No code change is needed at these call sites.
- **Items:** `src/App.tsx:173`, `src/App.tsx:177`, `src/App.tsx:197`, `src/App.tsx:198`, `src/__swaps__/screens/Swap/components/FlipButton.tsx:97`, `src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:75`, `src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:173`, `src/__swaps__/screens/Swap/components/TokenList/TokenToSellList.tsx:88`, `src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:8`, `src/__swaps__/screens/Swap/hooks/useSearchCurrencyLists.ts:161`, `src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438`, `src/__swaps__/screens/Swap/hooks/useSwapSettings.ts:14`, `src/__swaps__/screens/Swap/providers/SyncSwapStateAndSharedValues.tsx:111`, `src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023`, `src/app/error-boundary/ErrorBoundary.tsx:14`, `src/app/error-boundary/ErrorBoundary.tsx:21`, `src/components/Discover/DiscoverScreenContext.tsx:64`, `src/components/Discover/DiscoverSearch.tsx:117`, `src/components/Discover/DiscoverSearchInput.tsx:134`, `src/components/Discover/useTrackDiscoverScreenTime.ts:22`, `src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88`, `src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23`, `src/components/asset-list/RecyclerAssetList2/index.tsx:102`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174`, `src/components/cards/EthCard.tsx:74`, `src/components/cards/EthCard.tsx:90`, `src/components/cards/LearnCard.tsx:31`, `src/components/cards/ReceiveAssetsCard.tsx:47`
- **Verifier:** analytics/index.ts:29-244: `analytics` is the wrapper and the only PostHog user. Line 169 is the one direct client.capture ('Application Updated').

### F70. Legacy event and property names fail the naming pattern (R001, R007)
**Low.** _Confirmed by the verifier._
The event names are set as string constants in event.ts. Some are Title case with spaces ('Error during iCloud Backup') and some are dotted ('mined_transaction.assets_resolved'), so they don't match the snake_case `object_action` pattern. The R007 hits on `walletAddressHash` and `walletType` come from one place. The analytics wrapper adds both to every event, and the wallet-context type declares them in camelCase. That explains the 70 property hits, which are a single convention mismatch and not 70 separate mistakes. These are consistent, deliberate names, not bugs. The fix is to accept the codebase's existing convention.
- **Evidence:** `src/analytics/event.ts:66`, `src/analytics/event.ts:310`, `src/analytics/getWalletContext.ts:9`, `src/analytics/index.ts:147`
- **Fix:** Don't rename in place. Renaming breaks every report on the old name. Either set `_conventions.yaml` to allow this legacy casing and camelCase wrapper properties, or record the exceptions. If a rename is wanted, add the new event, mark the old one deprecated with `replaced_by`, and dual-fire for one release.
- **Items:** `Error during Google Drive Backup`, `Error during iCloud Backup`, `Error updating Backup status`, `iCloud not enabled`, `Ignore how to enable iCloud`, `Imported seed phrase`, `mined_transaction.assets_resolved`, `mined_transaction.assets_timed_out`, `network_status.offline`, `network_status.reconnected`, `pending_transaction.resolved`, `Scanned address QR code`, `walletAddressHash`, `walletType`
- **Verifier:** conventions.yaml event_pattern allows no dots or spaces, and index.ts:147-148 adds the camelCase walletAddressHash and walletType. This is a convention mismatch.

### F71. Generic and inconsistently named events reduce readability
**Low.** _Confirmed by the verifier._
`pressedButton` is a catch-all with free-text `buttonName` and `action`, so each new button creates a new value instead of an `entry_point`. The home-screen navigation events (`navigationSwap`, `navigationSend`, `navigationAddCash`, `navigationMyQrCode`) are separate events that all carry `category: 'home screen'`. By rule 3 they could be one action with an `entry_point`. Naming also mixes styles: 'Search Query', 'token_list', 'swaps.searched_for_token' and a camelCase `durationInMs`.
- **Evidence:** `src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157`, `src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174`, `src/components/Discover/useTrackDiscoverScreenTime.ts:22`
- **Fix:** For new tracking, use object_action snake_case and an `entry_point` property. Do not rename existing events in place. Where consolidation is wanted, deprecate with `replaced_by` and dual-fire for one release.
- **Items:** `pressedButton`, `navigation.add_cash`, `navigation.swap`, `navigation.send`, `navigation.my_qr_code`, `discover.tap_search`, `time_spent_on_discover_screen`
- **Verifier:** Pressed Button carries free-text buttonName and action, and the home-screen navigation events are separate events with `category: 'home screen'` (see the registry).

### F72. Trigger-order and add-to-position 'missing property' flags are scanner artefacts: the sibling success/failed calls are attributed to the wrong entry
**Low.** _Confirmed by the verifier._
Every flagged call sends exactly the properties its own event is typed for. The scanner matched each call to the wrong registry entry, probably because the success and failed calls sit in one try/catch. Line 281 is the perps.trigger_order.failed call, not trigger_order_created, so it is flagged for triggerPrice, leverage and positionValue, which only the success event carries. Line 71 is the cancel.failed call, not trigger_order_canceled. At PerpsAddToPositionSheet.tsx:214 the call is perps.open_position.canceled and at :242 it is added_to_position.failed. Neither sends entryPrice, positionSize or positionValue. The reverse also happens: errorMessage is flagged on the failed entries at :268, :60, :214 and :228, which are success or canceled calls. The success calls (CreateTriggerOrderBottomSheet.tsx:268, TriggerOrdersSection.tsx:60, PerpsAddToPositionSheet.tsx:228) send every property their entries require. The payload types in event.ts (lines 1232-1286) match the calls. The registry's own descriptions note the scanner folded these calls together. Nothing here is a code bug, and no claim can be made about arriving data because there is no snapshot.
- **Evidence:** `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281`, `src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242`, `src/analytics/event.ts:1232`
- **Fix:** Fix the scanner's call-to-entry matching so each call is checked against the event name it passes, not its neighbours in the same block. Then drop the false property_missing findings. No tracking change is needed. In the registry, mark properties 'Not sent by this event' as required: false. This is already done on the failed entries, but not on added_to_position (errorMessage) or the trigger-order entries.
- **Items:** `perps.trigger_order_created`, `perps.trigger_order.failed`, `perps.trigger_order_canceled`, `perps.trigger_order_cancel.failed`, `perps.added_to_position`, `perps.added_to_position.failed`
- **Verifier:** The success and failed calls each send their own typed properties (registry entries agree). The scanner attributed neighbouring calls to the wrong event.

### F73. Discover tab, section and carousel interactions double-fire: a product event plus a generic surface/placement interaction event
**Low.** _Confirmed by the verifier._
A tab press, a carousel scroll settle and a card press each send a Discover-specific event and then a second generic event (`surfaceInteraction` or `placementInteraction`) with mostly the same properties (section id, section title, display). Analysts who count both will count each action twice. `discoverCarouselScrolled` is a scroll event, which the design rules say to skip unless someone has a question it answers; it is debounced, which helps.
- **Evidence:** `src/features/discover/components/DiscoverHeader.tsx:134`, `src/features/discover/components/DiscoverHeader.tsx:139`, `src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17`, `src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:22`, `src/features/discover/utils/trackDiscoverCardPress.ts:28`, `src/features/discover/utils/trackDiscoverCardPress.ts:40`
- **Fix:** Decide which of the two is the report event for each action, and document the other as an engagement feed for placements. Raise the question on `discoverCarouselScrolled` before removing it; if removed, deprecate it.
- **Items:** `discoverTabPressed`, `discoverCarouselScrolled`, `discoverCardPressed`, `placementInteraction`, `surfaceInteraction`
- **Verifier:** DiscoverHeader.tsx:134-143 sends discoverTabPressed and then trackSurfaceInteraction for the same press.

### F74. walletAddressHash / walletType are added to every event by the analytics wrapper, so the R007 hits are about one shared source
**Low.** _Confirmed by the verifier._
No call passes these properties. `Analytics.track` merges `getDefaultMetadata()` into every capture and every identify (src/analytics/index.ts:147-148, 104 and 107). Drafting them as event properties on each event copies a global attribute onto each event entry. The camelCase names break the snake_case pattern, and they are undefined until `setWalletContext` runs. That is always the case for these two events: the probe fires at startup (App.tsx:177) and First App Open fires on first launch (App.tsx:198), before any wallet exists. Don't count on them for breakdowns of these events. Renaming them would break existing reports on every event.
- **Evidence:** `src/analytics/index.ts:147`, `src/analytics/index.ts:107`, `src/analytics/getWalletContext.ts:9`, `src/App.tsx:177`
- **Fix:** Remove them from per-event property lists and document them once as global/super properties in `_properties.yaml`. If the camelCase names are ever changed, add new snake_case names (`wallet_address_hash`, `wallet_type`) next to the old ones, and dual-write for a window. Don't rename in place.
- **Items:** `debug.identify_probe.walletAddressHash`, `debug.identify_probe.walletType`, `First App Open.walletAddressHash`, `First App Open.walletType`
- **Verifier:** index.ts:145-149 merges walletAddressHash and walletType into every capture and identify. _super_properties.yaml is empty and every entry repeats them.

### F75. Registry drafts list properties these events never send
**Low.** _Confirmed by the verifier._
The registry entries carry optional properties marked UNCONFIRMED that no call site sends. These are mode on phone_already_registered and phone_submit_failed, outcome and reason on phone_submitted, and reason and source on phone_verified. The event type map in event.ts declares only mode, reason or outcome, one per event. The extra entries look like the same call-site mix-up as in the first finding. They make the plan misleading, because an analyst would expect those breakdowns to exist.
- **Evidence:** `src/analytics/event.ts:573`, `src/analytics/event.ts:590`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/stores/verifyPhoneFlowStore.ts:127`
- **Fix:** Remove the UNCONFIRMED properties from the four entries. Keep only mode on phone_submitted and phone_verified, reason on phone_submit_failed and outcome on phone_already_registered. This is a registry-only edit: add a changelog line and make no code change.
- **Items:** `cash.phone_submitted`, `cash.phone_verified`, `cash.phone_submit_failed`, `cash.phone_already_registered`
- **Verifier:** The phone entries list UNCONFIRMED properties (mode, outcome, reason, source) that the calls don't send, as the registry rows themselves show.

### F76. WalletConnect property_missing results are calls matched to the wrong event, and the registry drafts carry stray properties
**Low.** _Confirmed by the verifier._
onSessionProposal.ts has four calls, each for a different event. Line 38 sends wcNewPairing with dappName, dappUrl and connector. Line 62 sends wcNewSessionRejected with dappName and dappUrl. Line 158 sends wcNewSessionApproved with dappName and dappUrl. Line 175 sends wcRequestFailed with type and reason. The scanner compared each event against the other calls in the file: 'connector' missing at 62/158/175, 'dappName/dappUrl' missing at 175, 'type/reason' missing at 38/62. Each event's own call has the properties it is supposed to have. The draft registry also lists properties on events that don't send them. Examples are connector, reason and type on the approved, rejected and pairing events. The descriptions of those properties say 'Not sent by this event'. They are marked optional, so they cause no errors. They do make the plan misleading. wc.failed_request also declares connector, dappName and dappUrl, and no call site sends them. The wc.failed_request description says 'Seven call sites' but lists six, and grep finds six.
- **Evidence:** `src/features/wallet-connect/handlers/onSessionProposal.ts:38`, `src/features/wallet-connect/handlers/onSessionProposal.ts:62`, `src/features/wallet-connect/handlers/onSessionProposal.ts:158`, `src/features/wallet-connect/handlers/onSessionProposal.ts:175`, `src/analytics/event.ts:661`
- **Fix:** Fix the scanner's call-to-event matching. Remove the properties that an event does not send (connector, reason and type on pairing, approved and rejected; connector, dappName and dappUrl on wc.failed_request) from the registry. Correct the call-site count in the wc.failed_request description. These are registry edits only. No event is renamed, so no deprecation is needed.
- **Items:** `New WalletConnect pairing`, `Rejected new WalletConnect session`, `Approved new WalletConnect session`, `wc.failed_request`
- **Verifier:** The four onSessionProposal calls (38/62/158/175) each send their own properties. The registry carries stray connector, reason and type rows, and the wc.failed_request description says "Seven call sites" but lists six.

### F77. `identify` is called with chart settings on every change, alongside a duplicate event
**Low.** _Confirmed by the verifier._
`ChartsTelemetry` calls `analytics.identify({ chartType })` and `analytics.track(chartTypeChanged, { chartType })` on each change, and does the same for `candleResolution`. Identify here uses the device id and writes the value to the profile, so every UI toggle becomes a profile write and an event. The events are also named in Title Case, unlike the dotted lower-case names elsewhere. This is a UI preference that a PM is unlikely to break a report down by.
- **Evidence:** `src/components/value-chart/Chart.tsx:271`, `src/components/value-chart/Chart.tsx:272`, `src/components/value-chart/Chart.tsx:282`, `src/components/value-chart/Chart.tsx:283`
- **Fix:** Keep the user property if you segment by chart type, and drop the matching event, or the reverse, if nobody can name the question it answers (rule 1). If you do keep both, rename the events via deprecate + `replaced_by`.
- **Items:** `Changed Chart Type`, `Changed Candle Resolution`, `chartType`, `candleResolution`
- **Verifier:** Chart.tsx:271-283 calls identify and track on each change.

### F78. QR scan outcomes are four separate events
**Low.** _Confirmed by the verifier._
The same action, a QR scan, is split by result type into four events. It should be one event with a property. The event names are also free text ('Scanned address QR code') and don't match the snake_case dotted style used by newer events such as `analytics_tracking.enabled`.
- **Evidence:** `src/hooks/useScanner.ts:57`, `src/hooks/useScanner.ts:78`, `src/hooks/useScanner.ts:105`, `src/analytics/event.ts:280`
- **Fix:** Add one `qr_code_scanned` event with `kind = address | profile | walletconnect | invalid`. Deprecate the four old events with `replaced_by` and dual-fire for one release.
- **Items:** `qrCodeScannedAddress`, `qrCodeScannedProfile`, `qrCodeScannedWalletConnect`, `qrCodeScannedInvalid`
- **Verifier:** The registry has four qrCodeScanned* events split by result type.

### F79. property_missing findings are a script artefact: failure calls were matched against the success entry (and vice versa)
**Low.** _Confirmed by the verifier._
The flagged lines are not calls of the entry they were checked against. claimAirdrop.ts:49 and :116 send rnbwAirdropClaimFailed ('rnbw_airdrop.claim.failed'), and claimRewards.ts:75 and :151 send rnbwRewardsClaimFailed. They were compared with the success entries, so claimId, claimedRnbw, decimals, pollAttempts and status look missing. The failure events correctly don't carry success-only fields. Going the other way, claimRewards.ts:123 is the success call (rnbwRewardsClaim) but was checked against the .failed entry, hence 'errorMessage missing'. The success call sends no errorMessage, as it should. The typed payloads in event.ts confirm the split. rnbw_airdrop.claim.failed has no registry entry at all, which probably caused the mix-up. The .failed registry entry also lists success-only properties (claimedRnbw, decimals, pollAttempts and others), with decimals and pollAttempts typed as string where they are numeric. These are stale draft values and should be removed.
- **Evidence:** `src/features/rnbw-rewards/utils/claimAirdrop.ts:49`, `src/features/rnbw-rewards/utils/claimAirdrop.ts:116`, `src/features/rnbw-rewards/utils/claimRewards.ts:75`, `src/features/rnbw-rewards/utils/claimRewards.ts:123`, `src/features/rnbw-rewards/utils/claimRewards.ts:151`, `src/analytics/event.ts:1443`
- **Fix:** Add a registry entry for rnbw_airdrop.claim.failed, with properties chainId, claimId?, status?, errorMessage, durationMs and platformRequestIds. Remove the success-only properties from rnbw_rewards.claim.failed. Fix the scanner's event-to-call matching so it compares each call to the entry for the event it sends. No change to the code's tracking calls is needed.
- **Items:** `rnbw_airdrop.claim`, `rnbw_rewards.claim`, `rnbw_rewards.claim.failed`
- **Verifier:** The misattribution holds (claimAirdrop.ts:49/116 are failure calls, claimRewards.ts:123 is the success call), and the stale string-typed decimals and pollAttempts on rnbw_rewards.claim.failed are real. One part is wrong: rnbw_airdrop.claim.failed already has a registry entry.

### F80. notification_permission.denied and .granted are missed when the permission request throws
**Low.** _Confirmed by the verifier._
The request is tracked before the OS prompt. If requestNotificationPermission throws, neither granted nor denied fires. The try has no catch, so the error is not logged and the finally block still navigates to the wallet. These cases look like drop-off between requested and its result.
- **Evidence:** `src/features/notifications/screens/NotificationPermissionScreen.tsx:65`, `src/features/notifications/screens/NotificationPermissionScreen.tsx:72`
- **Fix:** Add a catch branch that tracks the denied/error outcome, or log it, so every requested has a result.
- **Items:** `notification_permission.denied`, `notification_permission.granted`, `notification_permission.requested`
- **Verifier:** NotificationPermissionScreen.tsx:63-74 is try/finally with no catch, so a throw sends neither granted nor denied.

### F81. An invalid OTP code is reported as `client_error`, and refused sign-ins have no outcome event
**Low.** _Confirmed by the verifier._
In recovery, the server's `codeInvalid` verdict is sent as `cashPhoneVerifyFailed` with `reason: 'client_error'`. The server rejected it, so this mixes a user mistake in with real client errors in the `reason` breakdown. Separately, `cashSignInSubmitted` fires as the intent event. When the error is a cash-access refusal, none of succeeded, cancelled or failed is sent, so that attempt shows up as abandonment.
- **Evidence:** `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101`, `src/features/cash/services/cashSignInService.ts:53`, `src/features/cash/services/cashSignInService.ts:77`
- **Fix:** Use a distinct reason such as `code_invalid` for a rejected OTP. For refused sign-ins, either send a `failed` outcome with `failure_reason = access_refused`, or state in the event description that refusals are intentionally untracked.
- **Items:** `cashPhoneVerifyFailed`, `cashSignInSubmitted`
- **Verifier:** cashSignInService.ts:75-82: a cash-access refusal sends no outcome after cashSignInSubmitted. The registry confirms codeInvalid is sent as client_error.

### F82. Token launcher abandonment is only tracked when the user confirms discarding
**Low.** _Confirmed by the verifier._
`tokenLauncherAbandoned` is only sent in the discard-confirm branch. Exiting with no info entered returns early with no event. Abandonment is tracked only for one exit path, so drop-off is under-counted. `step_changed` is sent inside the store setter, so every `setStep` call anywhere fires it, including the REVIEW fallback after a failed creation, which looks like a step regression.
- **Evidence:** `src/screens/token-launcher/components/TokenLauncherHeader.tsx:45`, `src/screens/token-launcher/components/TokenLauncherHeader.tsx:59`, `src/screens/token-launcher/state/tokenLauncherStore.ts:292`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:112`
- **Fix:** Send the abandonment event on every exit with a `had_entered_info` property. Add a `from_step` property to `step_changed` so a retry after a failure can be told apart from forward progress.
- **Items:** `token_launcher.abandoned`, `token_launcher.step_changed`
- **Verifier:** TokenLauncherHeader.tsx:45-48 exits with no event when no info was entered. TokenLauncherFooter.tsx:112 calls setStep(REVIEW) after a failed or empty creation.

### F83. dynamic_name findings come from the test file and from screen(routeName)
**Low.** _Confirmed by the verifier._
In the test file, every flagged call passes analytics.event.<name> or a Routes constant to the wrapper. These are catalogued constants, not names built at runtime. These calls are unit-test assertions on the adapter and are not product tracking. In onNavigationStateChange.js, analytics.screen(routeName, ...) takes a route name from navigation state. The names come from the closed Routes set, but they are not literals, so the scanner cannot list them. The Routes set can still be listed from code, so these screen events are not unreadable tracking. The four duplicate hits on line 130 are one call.
- **Evidence:** `src/analytics/index.test.ts:72`, `src/analytics/index.test.ts:131`, `src/navigation/onNavigationStateChange.js:126-130`, `src/analytics/index.ts:91`
- **Fix:** Exclude *.test.ts from the scan. For screens, register the Routes set once as the source of screen names (the "Loaded A Screen" event) and mark the call as dynamic-but-enumerated.
- **Items:** `src/analytics/index.test.ts`, `src/navigation/onNavigationStateChange.js:130`
- **Verifier:** index.ts:91 shows analytics.screen takes a route constant. The test-file hits are assertions, not product tracking.

### F84. The tracking map has the firing sites swapped for these two events
**Low.** _Confirmed by the verifier._
`_map/error_boundary.yaml` lists both `onError` and `onReset` as fires_in for each event. In the code, `viewed` is sent only from `onError` and `reset` only from `onReset`. The path and trigger text are also wrong for `viewed`: its path ends at `onReset`. This makes the map misleading for any later impact analysis of this area, although the registry descriptions are correct.
- **Evidence:** `src/app/error-boundary/ErrorBoundary.tsx:13`, `src/app/error-boundary/ErrorBoundary.tsx:20`, `analytics/registry/_map/error_boundary.yaml:25`, `analytics/registry/_map/error_boundary.yaml:8`
- **Fix:** Edit the map through a reviewed PR: `error_boundary.viewed` fires_in `ErrorBoundary.tsx#onError` only, and `error_boundary.reset` fires_in `ErrorBoundary.tsx#onReset` only. The registry entries are unaffected.
- **Items:** `error_boundary.viewed`, `error_boundary.reset`
- **Verifier:** map/error_boundary.yaml:7-9 and 24-26 list both onError and onReset for each event, and the viewed path ends at onReset (38). ErrorBoundary.tsx:13-26 sends reset only from onReset and viewed only from onError.

### F85. Both performance events are sent only in production builds
**Low.** _Confirmed by the verifier._
PerformanceTracker.analyticsTrackingEnabled is initialised from IS_PROD, and trackMetric only calls analytics.track when it is true. Dev and test builds drop the event, and the debug console log is off by default. Because of this, a missing event in a non-production environment is expected behaviour and not a dead path. Without a data snapshot, nothing can be said about whether production data arrives.
- **Evidence:** `src/performance/tracking/index.ts:48`, `src/performance/tracking/index.ts:85`
- **Fix:** No code change. State the IS_PROD gating in the registry, as the descriptions already do, and verify arrival in production data once a snapshot exists.
- **Items:** `Performance Wallet Initialize Time`, `performance.report`
- **Verifier:** performance/tracking/index.ts:48 sets `analyticsTrackingEnabled = IS_PROD`, and line 85 gates the send on it.

### F86. perps.trigger_order_cancel.failed leaves out properties that the plan lists
**Low.** _Confirmed by the verifier._
The plan gives the failed event the same properties as the success event: leverage, positionValue and triggerPrice. The failed call sends only market, side, triggerOrderType, perpsBalance and errorMessage. All the missing values are available in the handler, because the position is loaded and the order has a trigger price. Failures can't be broken down by leverage, position size or trigger price the way successes can.
- **Evidence:** `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71`, `src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60`
- **Fix:** Add triggerPrice: Number(order.triggerPrice), leverage: position.leverage and positionValue: Number(position.value) to the failed call.
- **Items:** `perps.trigger_order_cancel.failed`, `perps.trigger_order_canceled`
- **Verifier:** TriggerOrdersSection.tsx:71-77 omits triggerPrice, leverage and positionValue, although all three are in scope (as at 60-68).

### F87. swaps.failed sends the raw execution error text
**Low.** _Confirmed by the verifier._
errorMessage is the free-text error returned by the swap rap. It is forwarded unchanged as a property, and the 'handled' sentinel is mixed in with real messages. Error strings from providers or RPC nodes can include addresses, amounts or request details. The type signature says string, but trackSwapEvent applies `errorMessage ?? null`.
- **Evidence:** `src/__swaps__/screens/Swap/providers/swap-provider.tsx:342`, `src/__swaps__/utils/trackSwapEvent.ts:49`
- **Fix:** Map the error to an enum `failure_reason` (for example handled, insufficient_sponsor_balance, unknown) and truncate or drop the raw text. This is a property change on an existing event. Add a new property name and dual-send it for one release. Don't retype errorMessage in place.
- **Items:** `swaps.failed`
- **Verifier:** swap-provider.tsx:342-347 forwards the raw errorMessage, and the 'handled' sentinel is mixed in with real messages.

### F88. EthCard fires card.pressed only when the ETH asset has a native value, so some taps go untracked
**Low.** _Confirmed by the verifier._
handleAssetPress returns early when ethAsset.native is null. That happens before both the navigation and the analytics.track call, so taps in that state aren't counted. Taps on the ETH card are therefore counted only after the asset price data has loaded. The same pattern means buy_button.pressed does not fire when the wallet is damaged, because the handler opens the error sheet and returns first. The registry descriptions already document both cases. Treat card.pressed as taps that did something, not as raw taps, when reading it.
- **Evidence:** `src/components/cards/EthCard.tsx:84`, `src/components/cards/EthCard.tsx:68`
- **Fix:** No code change is needed if 'taps that led somewhere' is the intended meaning. Keep that wording in the description and the question. If raw taps matter, move the track call before the guard. That would change the firing moment and needs a SEMANTICS: changelog line.
- **Items:** `card.pressed`
- **Verifier:** Both the registry entry and the cited lines show early returns before track: in EthCard when native is null, and when the wallet is damaged.

### F89. cash.phone_submitted description is incomplete: its call sites are in three files and the first is shared
**Low.** _Confirmed by the verifier._
The description lists three call sites, which matches the code. They are in different files than a reader would assume. The main one is advanceToChallenge in useSubmitPhoneFlow.ts, which signup, resume and recovery from the phone step all go through (the existing-account recovery choice reaches it via chooseRecovery). The `recovery` mode call is in verifyPhoneFlowStore.ts, and the `resume` mode call is in useSubmitReviewFlow.ts. The scanner's references only point at the shared file, which is probably why it matched the wrong calls.
- **Evidence:** `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68`, `src/features/cash/stores/verifyPhoneFlowStore.ts:106`, `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:124`
- **Fix:** Add the verifyPhoneFlowStore.ts and useSubmitReviewFlow.ts references to the registry entry, so scanning and drift checks see all three call sites.
- **Items:** `cash.phone_submitted`
- **Verifier:** The registry lists the three call sites, but the code refs only point at useSubmitPhoneFlow.ts.

### F90. Client-side 'Invalid amount' refusal is counted as a server failure on perps.added_to_position.failed
**Low.** _Confirmed by the verifier._
The sheet throws RainbowError('Invalid amount') when the amount is 0 or less. The catch branch then sends perps.added_to_position.failed with the error message, although no request was ever formed. This inflates the failure rate of the exchange call with user input errors. Rule 4 sends refusals like this to error_occurred.
- **Evidence:** `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:201`, `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242`
- **Fix:** Check the amount before the try block and send the refusal as error_occurred with feature, action and error_reason, or keep it here and add a failure_reason value that separates 'invalid_amount' from exchange errors. Changing what the failed event counts is a firing-moment change, so add a SEMANTICS changelog line.
- **Items:** `perps.added_to_position.failed`
- **Verifier:** PerpsAddToPositionSheet.tsx:201-203 throws 'Invalid amount' into the catch at 239-248, which sends added_to_position.failed.

### F91. Redeem failure loses the relayer transaction details when waiting for the relayer throws
**Low.** _Confirmed by the verifier._
`tx` is only assigned when waitForRelayerTransaction returns, at line 44. If the wait throws after the transaction was submitted, the catch at line 51 passes `tx` as undefined. The failure event then has no transactionHash, transactionId or transactionState at line 89. That is the case where the state would be most useful for diagnosis.
- **Evidence:** `src/features/polymarket/utils/redeemPosition.ts:44`, `src/features/polymarket/utils/redeemPosition.ts:89`
- **Fix:** Capture the relayer response from submitTradingWalletTransaction before waiting. On failure, send its transaction id and state as optional properties. This is additive.
- **Items:** `predictions.redeem_position.failed`
- **Verifier:** redeemPosition.ts:40-51: `tx` is only set after waitForRelayerTransaction, so the failure event at 89-91 has no tx details when the wait throws.

### F92. watch_wallet fires before the import, on an intent-only moment, and the `addressOrEnsName` value is inconsistent
**Low.** _Confirmed by the verifier._
The event fires before startImportProfile, so it records intent with no matching outcome event. Failures or cancellation of the profile sheet are not measurable. For raw addresses, `addressOrEnsName` is the reverse-ENS result, which can be empty or null, or the address itself if the lookup threw. So the property mixes name, address and empty values, which makes breakdowns by entry type unreliable.
- **Evidence:** `src/hooks/useImportingWallet.ts:233`, `src/hooks/useImportingWallet.ts:213`, `src/hooks/useImportingWallet.ts:239`
- **Fix:** Add an `input_kind` property (ens | unstoppable | address | derived) to this event. Track the outcome through the existing import success path, with `status`. Both are additive, so no breaking change.
- **Items:** `watch_wallet`
- **Verifier:** useImportingWallet.ts:232-239 sends watch_wallet before startImportProfile, and `ens` defaults to the input (211) or '' from the reverse lookup.

### F93. Registry entry doesn't match what the call sends or how it behaves
**Low.** _Confirmed by the verifier._
The entry lists `address`, `addressOrEnsName` and `type`, which this call never sends. They belong to watchWallet and the imported-wallet event. The `ens` description says it equals the input address when no name exists. In fact fetchReverseRecord swallows every error and returns '' when there is no reverse record, so `ens` is an empty string in that case. The note 'not fired if the lookup throws' also doesn't hold. fetchReverseRecord never throws, so the catch around it is effectively dead and the event fires for every valid 0x address. The inner `ens !== input` check is therefore always true when `ens` is non-empty. The name says read-only, but the event fires for any plain address, including type 'import'.
- **Evidence:** `src/features/ens/utils/handlers.ts:839`, `src/hooks/useImportingWallet.ts:213`, `src/hooks/useImportingWallet.ts:222`
- **Fix:** Edit the registry entry. Keep only `ens` and `input` (plus the auto-added wallet context properties). Document `ens` as empty when no reverse record exists. State that the event fires for every valid plain address, whatever `type` is. Don't rename the event in place. If a clearer name is wanted, add a new event, deprecate this one with `replaced_by`, and dual-fire for one release.
- **Items:** `Show wallet profile modal for read only wallet`
- **Verifier:** handlers.ts:839-848: fetchReverseRecord swallows every error and returns '', so the catch is dead, the event fires for every valid address, and `ens` is empty when there is no reverse record.

### F94. Temporary debug probe event still ships on every launch
**Low.** _Confirmed by the verifier._
The code comment says to remove the paired probe event after FEPLAT-67 wraps up. It fires on every app start, and it duplicates the installSource user property as an event. If the investigation is closed, this is noise with no question behind it.
- **Evidence:** `src/App.tsx:174`, `src/App.tsx:177`
- **Fix:** If FEPLAT-67 is done, remove the call and mark the event deprecated with a `deprecated_reason`. Keep the entry, per the removal rules.
- **Items:** `debug.identify_probe`
- **Verifier:** The comment at redux/settings.ts:119-121 says to remove the probe after FEPLAT-67, and it fires on startup and settings load.

### F95. Generic `cardPressed` event used for ENS cards, with the card identity in a free-text property
**Low.** _Confirmed by the verifier._
ENSCreateProfileCard and ENSSearchCard send the same `cardPressed` event and tell themselves apart only by a hard-coded `cardName` string. `routeName` and `cardType` are also sent. That fits rule 3 (one event, a property), so it is not a problem in itself. But the values are free-text literals in the components, so a rename of a card splits the breakdown.
- **Evidence:** `src/features/ens/components/ENSCreateProfileCard.tsx:51`, `src/features/ens/components/ENSSearchCard.tsx:54`
- **Fix:** Define `cardName` as a typed enum in `@/analytics/event` and list the values in the registry. No event change.
- **Items:** `cardPressed`
- **Verifier:** The registry shows cardName as hard-coded literals per component (card.pressed).

### F96. Performance metrics use event names chosen at runtime
**Low.** _Confirmed by the verifier._
trackMetric passes whatever `metric` it receives straight to analytics.track. The scanner cannot see the event names at that call site, so they are not in the plan. This is how dynamic names enter the project. The send is also gated by the `analyticsTrackingEnabled` flag, so whether these events arrive depends on a runtime setting.
- **Evidence:** `src/performance/tracking/index.ts:78`, `src/performance/tracking/index.ts:85`
- **Fix:** List every PerformanceEvent value in the registry, or document this file as an adapter. Consider one `performance_metric` event with a `metric` property.
- **Items:** `trackMetric`
- **Verifier:** performance/tracking/index.ts:78-86 passes the `metric` name through, gated by analyticsTrackingEnabled.

### F97. Token launcher share event fires after the share sheet closes, whether or not anything was shared, and carries the deeplink URL
**Low.** _Confirmed by the verifier._
The event is sent after `await Share.share(...)` resolves. That happens on dismiss as well as on a real share, so it counts completed share-sheet sessions and not presses. The full URL is attached on every event, which is a high-cardinality value. `address` is already sent, so the URL adds nothing.
- **Evidence:** `src/screens/token-launcher/components/TokenLauncherFooter.tsx:203`, `src/screens/token-launcher/components/TokenLauncherFooter.tsx:212`
- **Fix:** Fire on press, before `Share.share`, or read the result's `action` and send `status = shared | dismissed`. Remove `url`. If the firing moment changes, add a `SEMANTICS:` changelog line.
- **Items:** `token_launcher.share_pressed`
- **Verifier:** TokenLauncherFooter.tsx:203-212 tracks after `await Share.share` with no result check and sends `url`.

### F98. Leaderboard press sends the user's ENS name as an event property
**Low.** _Confirmed by the verifier._
`ens` is another person's public name, but it is identity-like data. It is high-cardinality and can't be broken down. The event carries no `rank` property, which is what someone would break the press count by.
- **Evidence:** `src/screens/rewards/components/RewardsLeaderboardItem.tsx:81`
- **Fix:** Remove `ens` and send `has_ens` and `rank` instead. The change alters the property set, so go through the modify path in `impact-analysis.md`.
- **Items:** `rewards.pressed_leaderboard_item`
- **Verifier:** The registry shows the leaderboard press sends `ens` and no rank.

### F99. qr_code.viewed fires on the tap, not when the QR code is shown
**Low.** _Confirmed by the verifier._
The event is sent in the tap handler, just before navigate(Routes.RECEIVE_MODAL). The name says "viewed", but the code only records an attempt to open the modal. The tap and the modal opening happen together, so the count is close to real views today. If the navigation is blocked or changed later, the count will drift from real views. The same tap is the only way to fire it. Copying the address does not fire it.
- **Evidence:** `src/components/cards/ReceiveAssetsCard.tsx:47`, `src/components/cards/ReceiveAssetsCard.tsx:50`
- **Fix:** Either treat it as a tap and rename it to something like qr_code.pressed. Or send it from the receive modal when it mounts, so it counts real views.
- **Items:** `qr_code.viewed`
- **Verifier:** Per the registry and the cited lines, the event is tracked in the tap handler before navigate(RECEIVE_MODAL).

### F100. app_icon_unlock_sheet.cta_pressed is sent after the navigation and a 500 ms wait, not on the press
**Low.** _Confirmed by the verifier._
The handler calls goBack and navigate, then waits 500 ms, navigates again, and only then tracks. The name and plan describe a press, so if the second navigation throws or the app is backgrounded during the delay, the tap is never counted. Presses are undercounted and the timing is shifted by about half a second.
- **Evidence:** `src/features/app-icon/screens/AppIconUnlockSheet.tsx:37`, `src/features/app-icon/screens/AppIconUnlockSheet.tsx:39`
- **Fix:** Move the analytics.track call to the top of navigateToAppIconSettings, before goBack.
- **Items:** `app_icon_unlock_sheet.cta_pressed`
- **Verifier:** AppIconUnlockSheet.tsx:34-39 does goBack, navigate, `await delay(500)` and navigate before tracking.

### F101. Backup Complete fires for single-wallet backups too, not only 'all wallets'
**Low.** _Confirmed by the verifier._
The plan says it fires from the onSuccess of backupAllWalletsToCloud. The same onSuccess is also passed to walletCloudBackup when a walletId is set, and that hook calls it after setWalletBackedUp. So the event covers both branches, with no property to tell them apart. The failure side is split: backupError for all wallets, errorDuringICloudBackup or errorDuringGoogleDriveBackup for one wallet. A completed-versus-error rate for all-wallet backups is therefore wrong.
- **Evidence:** `src/features/backup/hooks/useCreateBackup.ts:65`, `src/features/backup/hooks/useCreateBackup.ts:119`, `src/hooks/useWalletCloudBackup.ts:146`
- **Fix:** Add a property such as scope (all | single) to Backup Complete, or update the plan to say it covers both branches. Use one failure event for both.
- **Items:** `Backup Complete`
- **Verifier:** useCreateBackup.ts:119-125 passes the same onSuccess (which sends backupComplete at 65) to walletCloudBackup, which calls it at useWalletCloudBackup.ts:146.

### F102. Set App Icon is sent before the icon change is applied, on every tap
**Low.** _Confirmed by the verifier._
onSelectIcon sends the event first and then calls settingsChangeAppIcon. That call awaits the OS icon change and logs an error in its catch, so the change can fail. The event is also sent when the person taps the icon that is already selected. Counts of Set App Icon therefore include failed changes and no-op taps. They are attempts, not icons actually set.
- **Evidence:** `src/features/app-icon/components/AppIconSettingsSection.tsx:32`, `src/features/app-icon/components/AppIconSettingsSection.tsx:33`, `src/redux/settings.ts:174`
- **Fix:** Skip the event when icon equals the current appIcon. If the question is which icon people end up with, send it after changeIcon resolves in settingsChangeAppIcon. Otherwise keep it as an intent event and add a matching outcome.
- **Items:** `Set App Icon`
- **Verifier:** AppIconSettingsSection.tsx:32-33 tracks before settingsChangeAppIcon, with no same-icon check, and that call can fail in its own catch (redux/settings.ts:173).

### F103. Edited ENS records can fire several times per rap with no step in the payload
**Low.** _Confirmed by the verifier._
Four case branches send the same event. A rap with several record steps (for example set text, then set address) sends it once per step. Nothing in the payload says which step it was, so the event count is not the count of edit actions and the steps can't be separated.
- **Evidence:** `src/features/ens/raps/actions.ts:400`, `src/features/ens/raps/actions.ts:434`, `src/features/ens/raps/actions.ts:440`, `src/features/ens/raps/actions.ts:446`
- **Fix:** Send it once per rap after the record steps, or add a step type property (text, contenthash, address, multicall) and count distinct raps.
- **Items:** `Edited ENS records`
- **Verifier:** actions.ts:400, 434, 440 and 446 each send ensEditedRecords with only `category`.

### F104. perps.trade_details.share_pressed fires only after the share dialog resolves
**Low.** _Confirmed by the verifier._
The name and intent say the Share button was pressed, but the call sits after await RNShare.open. A press where the image capture returns nothing, or where the share throws, sends nothing. Cancelled shares are sent with shared=false. The number of presses is therefore undercounted, and the name implies a different moment than the code has.
- **Evidence:** `src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73`
- **Fix:** Send the press event at the top of handleShare, and report the result (shared, success) on a separate event or on a property of an outcome event.
- **Items:** `perps.trade_details.share_pressed`
- **Verifier:** PerpsTradeDetailsSheet.tsx:61-73 returns early when there is no image, and the track call comes after `await RNShare.open`.

### F105. Viewed ENS profile sends a hardcoded 'from' value
**Low.** _Confirmed by the verifier._
onAvatarViewProfile always sends from = 'Transaction list'. The hook is used by AvatarCircle, which is reachable from several screens (wallet, profile sheets and others). Any 'from' breakdown will label every view as the transaction list, whatever screen the tap came from.
- **Evidence:** `src/hooks/useOnAvatarPress.ts:111-115`
- **Fix:** Pass the real source screen or entry point into the hook and send it as 'from'. Alternatively, remove the property if it can't be filled correctly.
- **Items:** `Viewed ENS profile`
- **Verifier:** useOnAvatarPress.ts:111-115 hard-codes `from: 'Transaction list'`.

### F106. Wallet init timer is never cleared on early-return paths, and concurrent runs share one timer key
**Low.** _Confirmed by the verifier._
The timer is keyed only by the event name, and `Timer.start` overwrites any existing entry. The `shouldCancel()` returns and the nil-address return leave the timer running. They don't call `clearMeasure`. Only the throw path clears it. If two initializeWallet runs overlap (a switch while one is still running, or a cancelled run followed by a new one), the second `startMeasuring` resets the start time. The first run's `finishMeasuring` then consumes the second run's timer, and the second run finds no timer and sends nothing. The duration can be too short, and one run can go unreported. This is limited to rare overlap or cancel cases. The event does fire on success, as the plan says.
- **Evidence:** `src/state/wallets/initializeWallet.ts:70`, `src/state/wallets/initializeWallet.ts:98`, `src/state/wallets/initializeWallet.ts:156`, `src/performance/timer.ts:7`
- **Fix:** Clear the measure on every early return, for example with a `finally` that calls `clearMeasure` when the run did not finish. Or key the timer by a per-run id.
- **Items:** `Performance Wallet Initialize Time`
- **Verifier:** timer.ts:6-11 keys the timer by name and overwrites it. The shouldCancel() returns (initializeWallet.ts:98, 112, 125, 141, 146, 161) and the nil-address return (156) never clear it.

### F107. Tapped "Cancel" after editing wallet fires on any close with an empty name, not on a Cancel tap
**Low.** _Confirmed by the verifier._
The event is sent from the else branch of 'if (name)' in onCloseModal, so it fires whenever the modal closes with an empty name, including after saving and after dismissing by swipe. updateAccountInfo has already run with an undefined label by then, so the label is cleared even though the event says the person cancelled. The count is closes-with-empty-name, not cancels.
- **Evidence:** `src/screens/change-wallet/ChangeWalletSheet.tsx:259`, `src/screens/change-wallet/ChangeWalletSheet.tsx:267`
- **Fix:** Treat this as 'wallet edit closed without a name' (or send the cancel event only from a real Cancel action). Skip the label update when the name is empty.
- **Items:** `Tapped "Cancel" after editing wallet`
- **Verifier:** ChangeWalletSheet.tsx:257-268 runs updateAccountInfo with an undefined label, then sends tappedCancelEditingWallet whenever the name is empty.

### F108. Tapped Edit fires on both entering and leaving edit mode with nothing to tell them apart
**Low.** _Confirmed by the verifier._
onPressEditMode sends the event and then toggles editMode. Each tap is counted the same whether it starts or ends editing. A count of 'edit started' is inflated by the exits, and the event also fires when the tap only dismisses the hint tooltip.
- **Evidence:** `src/screens/change-wallet/ChangeWalletSheet.tsx:411`, `src/screens/change-wallet/ChangeWalletSheet.tsx:415`
- **Fix:** Send the new mode as a property (for example is_editing = !editMode), or split the event into entered and exited.
- **Items:** `Tapped Edit`
- **Verifier:** ChangeWalletSheet.tsx:410-415 tracks and then toggles editMode, with nothing to tell entering from leaving.

### F109. debug.identify_probe (currency) can fire more than once per app session
**Low.** _Confirmed by the verifier._
The currency probe sits inside settingsLoadState, which is reached through loadSettingsData. loadSettingsData runs once at startup and again in initializeWallet for every existing wallet that is not new or being imported. So the probe with probe='currency' fires at startup and again on each wallet initialization, such as switching wallets. The plan describes it as firing once in settingsLoadState. The event is temporary and diagnostic, so it only inflates probe counts and could confuse the delivery comparison it was built for.
- **Evidence:** `src/redux/settings.ts:121`, `src/state/wallets/initializeWallet.ts:160`, `src/App.tsx:179`
- **Fix:** When comparing delivery, count probe='currency' per session, not per event. Or send the probe only on the startup load. Remove the event once FEPLAT-67 is done.
- **Items:** `debug.identify_probe`
- **Verifier:** loadSettingsData.ts:12 dispatches settingsLoadState (which holds the probe at redux/settings.ts:121), and initializeWallet.ts:159-160 calls it for every existing wallet.

### F110. performance.time_to_sign also fires when the person cancels or dismisses the dapp-request sheet
**Low.** _Confirmed by the verifier._
In SignTransactionSheet the end-of-flow step is closeScreen, a SheetDismissal operation. onCancel calls it with canceled = true, and the confirm path calls it too. A cancelled request therefore sends a time_to_sign for a flow that never signed. A short cancel time pulls down the average time to sign for dapp requests. There is no property to tell cancelled, failed and signed flows apart.
- **Evidence:** `src/features/dapp-request/screens/SignTransactionSheet.tsx:265`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:282`, `src/features/dapp-request/screens/SignTransactionSheet.tsx:301`
- **Fix:** Add an outcome property (signed, failed or cancelled) to time_to_sign, or mark the dismissal wrapper as end of flow only on the confirm path.
- **Items:** `performance.time_to_sign`
- **Verifier:** SignTransactionSheet.tsx:301: onCancel calls closeScreen(true), the end-of-flow SheetDismissal, so cancelled requests also send time_to_sign with no outcome property.

### Smaller items (found by the scripts, not explained by an agent)

- `src/features/app-icon/components/AppIconSettingsSection.tsx:32` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/app-icon/components/AppIconSettingsSection.tsx:32)
- `src/features/backup/components/BackupCloudStep.tsx:58` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/backup/components/BackupCloudStep.tsx:58)
- `src/features/backup/components/BackupSheetSection.tsx:56` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/backup/components/BackupSheetSection.tsx:56)
- `src/features/cash/screens/add-cash-sheet/ReauthenticateContent.tsx:51` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/add-cash-sheet/ReauthenticateContent.tsx:51)
- `src/features/cash/screens/add-wallet-sheet/AddWalletSheet.tsx:87` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/add-wallet-sheet/AddWalletSheet.tsx:87)
- `src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36)
- `src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40)
- `src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47)
- `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:134` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:134)
- `src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:154` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:154)
- `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:176` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:176)
- `src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:227` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:227)
- `src/features/cash/services/kycStatusService.ts:22` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/cash/services/kycStatusService.ts:22)
- `src/features/dapp-request/hooks/useTransactionSetup.ts:55` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/dapp-request/hooks/useTransactionSetup.ts:55)
- `src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228)
- `src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73)
- `src/features/perps/withdrawalConfig.ts:48` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/perps/withdrawalConfig.ts:48)
- `src/features/transfer/screens/SendSheet.tsx:554` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/transfer/screens/SendSheet.tsx:554)
- `src/features/transfer/utils/trackSentTransaction.ts:32` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/transfer/utils/trackSentTransaction.ts:32)
- `src/features/wallet-connect/components/WalletConnectV2ListItem.tsx:112` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/features/wallet-connect/components/WalletConnectV2ListItem.tsx:112)
- `src/hooks/useHiddenTokens.ts:34` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useHiddenTokens.ts:34)
- `src/hooks/useHiddenTokens.ts:73` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useHiddenTokens.ts:73)
- `src/hooks/useIsOffline.ts:39` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useIsOffline.ts:39)
- `src/hooks/useIsOffline.ts:47` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useIsOffline.ts:47)
- `src/hooks/useIsOffline.ts:52` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useIsOffline.ts:52)
- `src/hooks/useRefreshAccountData.ts:59` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useRefreshAccountData.ts:59)
- `src/hooks/useWalletCohort.ts:64` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useWalletCohort.ts:64)
- `src/hooks/useWatchPendingTxs.ts:92` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/hooks/useWatchPendingTxs.ts:92)
- `src/model/wallet.ts:498` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/model/wallet.ts:498)
- `src/performance/tracking/index.ts:86` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/performance/tracking/index.ts:86)
- `src/screens/AddWalletSheet.tsx:59` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/AddWalletSheet.tsx:59)
- `src/screens/AddWalletSheet.tsx:70` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/AddWalletSheet.tsx:70)
- `src/screens/AddWalletSheet.tsx:71` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/AddWalletSheet.tsx:71)
- `src/screens/FiatOnRamp/ProviderListItem.tsx:35` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/FiatOnRamp/ProviderListItem.tsx:35)
- `src/screens/LearnWebViewScreen.tsx:51` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/LearnWebViewScreen.tsx:51)
- `src/screens/change-wallet/ChangeWalletSheet.tsx:267` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/change-wallet/ChangeWalletSheet.tsx:267)
- `src/screens/change-wallet/ChangeWalletSheet.tsx:286` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/change-wallet/ChangeWalletSheet.tsx:286)
- `src/screens/change-wallet/ChangeWalletSheet.tsx:295` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/change-wallet/ChangeWalletSheet.tsx:295)
- `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:95` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:95)
- `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:157` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:157)
- `src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:175` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:175)
- `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:440` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:440)
- `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:491` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:491)
- `src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:534` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:534)
- `src/screens/rewards/components/RewardsStats.tsx:42` [direct_sdk]: call the tracking wrapper instead (the tracking profile lists them) (src/screens/rewards/components/RewardsStats.tsx:42)
- `src/analytics/index.test.ts:86` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:86)
- `src/analytics/index.test.ts:101` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:101)
- `src/analytics/index.test.ts:146` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:146)
- `src/analytics/index.test.ts:148` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:148)
- `src/analytics/index.test.ts:148` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:148)
- `src/analytics/index.test.ts:179` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:179)
- `src/analytics/index.test.ts:199` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:199)
- `src/analytics/index.test.ts:233` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:233)
- `src/analytics/index.test.ts:234` [dynamic_name]: event names must be literals (src/analytics/index.test.ts:234)
- `app_icon_unlock_sheet.cta_pressed, app_icon_unlock_sheet.viewed, Approved new WalletConnect session, Backup Complete, backup.error, BackupSheet shown, browser.add_favorite, browser.tap_favorite, browser.trending_dapp_pressed, cash.amount_entered, cash.buy_completed, cash.buy_failed` [validate R001]: Event name does not match the naming pattern (115)
- `add_wallet_flow.started.isFirstWallet, add_wallet_flow.started.walletAddressHash, add_wallet_flow.started.walletType, analytics_tracking.disabled.walletAddressHash, analytics_tracking.disabled.walletType, analytics_tracking.enabled.walletAddressHash, analytics_tracking.enabled.walletType, Changed language.walletAddressHash, Changed language.walletType, Changed native currency.walletAddressHash, Changed native currency.walletType, Changed network.chainId` [validate R007]: Property name does not match the naming pattern (138)
- `app_icon_unlock_sheet.cta_pressed.appIcon, app_icon_unlock_sheet.cta_pressed.walletAddressHash, app_icon_unlock_sheet.cta_pressed.walletType, app_icon_unlock_sheet.viewed.appIcon, app_icon_unlock_sheet.viewed.walletAddressHash, app_icon_unlock_sheet.viewed.walletType, Approved new WalletConnect session.dappName, Approved new WalletConnect session.dappUrl, Approved new WalletConnect session.walletAddressHash, Approved new WalletConnect session.walletType, Backup Complete.walletAddressHash, Backup Complete.walletType` [validate R007]: Property name does not match the naming pattern (501)
- `appIcon, branchCampaign, branchReferrer, branchReferringLink, candleResolution, chartType, claimablesAmount, claimablesUSDValue, enabledTestnets, hardwareAccounts, hasImported, importedPrivateKeys` [validate R007]: Property name does not match the naming pattern (35)
- `buy_button.pressed.cardName, buy_button.pressed.cardType, buy_button.pressed.componentName, buy_button.pressed.routeName, buy_button.pressed.walletAddressHash, buy_button.pressed.walletType, card.pressed.cardName, card.pressed.cardType, card.pressed.componentName, card.pressed.routeName, card.pressed.walletAddressHash, card.pressed.walletType` [validate R007]: Property name does not match the naming pattern (14)
- `Changed Candle Resolution.candleResolution, Changed Candle Resolution.chartType, Changed Candle Resolution.walletAddressHash, Changed Candle Resolution.walletType, Changed Chart Type.candleResolution, Changed Chart Type.chartType, Changed Chart Type.walletAddressHash, Changed Chart Type.walletType` [validate R007]: Property name does not match the naming pattern (8)
- `claimable_panel.opened.chainId, claimable_panel.opened.claimableId, claimable_panel.opened.claimableType, claimable_panel.opened.usdValue, claimable_panel.opened.walletAddressHash, claimable_panel.opened.walletType, Pressed Button.buttonName, Pressed Button.walletAddressHash, Pressed Button.walletType, Tapped "Add Cash".walletAddressHash, Tapped "Add Cash".walletType, Tapped "My QR Code".walletAddressHash` [validate R007]: Property name does not match the naming pattern (17)
- `Error during Google Drive Backup.walletAddressHash, Error during Google Drive Backup.walletType, Error during iCloud Backup.walletAddressHash, Error during iCloud Backup.walletType, Error updating Backup status.walletAddressHash, Error updating Backup status.walletType, iCloud not enabled.walletAddressHash, iCloud not enabled.walletType, Ignore how to enable iCloud.walletAddressHash, Ignore how to enable iCloud.walletType, Imported seed phrase.addressOrEnsName, Imported seed phrase.walletAddressHash` [validate R007]: Property name does not match the naming pattern (70)
- `insufficient_native_asset_for_action.nativeAssetSymbol, insufficient_native_asset_for_action.walletAddressHash, insufficient_native_asset_for_action.walletType, swaps.changed_chain_id.chainId, swaps.changed_chain_id.inputAsset, swaps.changed_chain_id.walletAddressHash, swaps.changed_chain_id.walletType, swaps.failed.degenMode, swaps.failed.errorMessage, swaps.failed.inputAssetAddress, swaps.failed.inputAssetAmount, swaps.failed.inputAssetChainId` [validate R007]: Property name does not match the naming pattern (90)
- `Performance Wallet Initialize Time.durationInMs, Performance Wallet Initialize Time.performanceTrackingVersion, Performance Wallet Initialize Time.walletAddressHash, Performance Wallet Initialize Time.walletType, performance.report.durationInMs, performance.report.performanceTrackingVersion, performance.report.walletAddressHash, performance.report.walletType` [validate R007]: Property name does not match the naming pattern (8)
- `performance.time_to_sign.completedAt, performance.time_to_sign.elapsedTime, performance.time_to_sign.endTime, performance.time_to_sign.resultWasNullish, performance.time_to_sign.startTime, performance.time_to_sign.timeToCompletion, performance.time_to_sign.totalElapsedTime, performance.time_to_sign.walletAddressHash, performance.time_to_sign.walletType, performance.time_to_sign.operation.completedAt, performance.time_to_sign.operation.elapsedTime, performance.time_to_sign.operation.endTime` [validate R007]: Property name does not match the naming pattern (21)
- `wallet_create.failed.walletAddressHash, wallet_create.failed.walletType, wallet_create.succeeded.walletAddressHash, wallet_create.succeeded.walletType` [validate R007]: Property name does not match the naming pattern (4)
- `predictions.order_match.failed.status, predictions.place_order.status, rnbw_airdrop.claim.status, rnbw_airdrop.claim.failed.status, rnbw_rewards.claim.status, rnbw_rewards.claim.failed.status` [validate R012]: status without failure_reason (6)
- `*.amount, *.chainId, *.decimals, *.destination, *.no_icon, *.no_price, *.pnl, *.pollAttempts, *.total_tokens, *.usdValue, *.waitingTime` [validate R013]: Same property name has different data types (11)

## What wasn't checked

- install:explain:model (explain): model: wallet_create.failed.walletAddressHash, wallet_create.failed.walletType, wallet_create.succeeded.walletAddressHash, wallet_create.succeeded.walletType. too large for one answer, even alone.

## Migration plan

Areas in order: money and sign-up first, then core actions, then the rest. Each area is one fix PR (opened from the audit's findings), through the PR check with both human gates.

1. **features** (one PR)
   - **rename** `app_icon_unlock_sheet.cta_pressed` → `app_icon_unlock_sheet_cta_pressed`: dot breaks the snake_case naming pattern; keep the old name until dashboards are checked (src/features/app-icon/screens/AppIconUnlockSheet.tsx:39)
   - **rename** `app_icon_unlock_sheet.viewed` → `app_icon_unlock_sheet_viewed`: dot breaks the naming pattern (src/features/app-icon/screens/AppIconUnlockSheet.tsx:43)
   - **rename** `Approved new WalletConnect session` → `wc_session_approved`: mixed-case sentence name breaks the snake_case pattern (src/features/wallet-connect/handlers/onSessionProposal.ts:158)
   - **split status** `Rejected new WalletConnect session` → `wc_session_approved`: approved and rejected are one decision on one proposal. Fold them into one outcome event with status = approved | rejected (and a failure reason for the invalid-namespace case), so the funnel from pairing keeps its drop-off. (src/features/wallet-connect/handlers/onSessionProposal.ts:62)
   - **rename** `Backup Complete` → `backup_completed`: title-case name breaks the snake_case pattern (src/features/backup/hooks/useCreateBackup.ts:65)
   - **split status** `backup.error` → `backup_completed`: a failure event next to the success event (rule 4). Fold into the outcome event as status = success | failed plus failure_reason, so backup success rate needs no union. (src/features/backup/backup.ts:187)
   - **rename** `BackupSheet shown` → `backup_sheet_viewed`: mixed-case name breaks the snake_case pattern (src/features/backup/components/BackupSheetSection.tsx:56)
   - **property** `BackupSheet shown.label`: category is the constant 'backup' on every backup event, so it carries no information. label means the sheet type on this event but the cloud platform on others. Use distinct property names (e.g. sheet_type, cloud_platform). (src/features/backup/components/BackupSheetSection.tsx:56; src/features/backup/hooks/useCreateBackup.ts:65)
   - **rename** `browser.add_favorite` → `browser_favorite_added`: dot breaks the naming pattern (src/features/dapp-browser/stores/favoriteDappsStore.ts:50)
   - **rename** `browser.tap_favorite` → `browser_favorite_pressed`: dot breaks the naming pattern (src/features/dapp-browser/components/Homepage.tsx:226)
   - **rename** `browser.trending_dapp_pressed` → `browser_trending_dapp_pressed`: dot breaks the naming pattern (src/features/dapp-browser/components/Homepage.tsx:334)
   - **other** `browser.trending_dapp_pressed`: the `index && analytics.track(...)` guard skips index 0, so taps on the first tile are never counted. Use `index !== undefined`. (src/features/dapp-browser/components/Homepage.tsx:332-340)
   - **rename** `cash.amount_entered` → `cash_amount_entered`: dot breaks the naming pattern. The name also suggests keystroke entry, but it fires once on Hold to Add, so a name like cash_deposit_confirmed would describe it better. (src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477)
   - **rename** `cash.buy_submitted` → `cash_buy_submitted`: dot breaks the naming pattern (src/features/cash/stores/cashBuyOrderStore.ts:263)
   - **rename** `cash.buy_completed` → `cash_buy_completed`: dot breaks the naming pattern (src/features/cash/stores/cashBuyOrderStore.ts:123)
   - **split status** `cash.buy_failed` → `cash_buy_completed`: a _failed event next to the outcome event (rule 4). Fold into cash_buy_completed with status = success | failed, plus failure_reason and error_code. Failure rate then needs no union. (src/features/cash/stores/cashBuyOrderStore.ts:146)
   - **add pair** `cash.buy_submitted`: submitted has no orderId, so it can't be joined to its completed/failed outcome per order. Send the orderId (spec.id) on submitted, and on completed (which lacks it). (src/features/cash/stores/cashBuyOrderStore.ts:263)
   - **split status** `cash.card_link_failed` → `cash_card_linked`: a failure event next to the success event cash.card_linked. Fold into one outcome event with status = success | failed and failure_reason. (src/features/cash/stores/cardLinkFlowStore.ts:58)
   - **rename** `cash.card_linked` → `cash_card_linked`: the dot breaks the naming pattern (^[a-z][a-z0-9]*(_[a-z0-9]+)+$); no data is arriving, so dual-fire only if a dashboard uses it (src/analytics/event.ts:148)
   - **rename** `cash.deposit_intro_viewed` → `cash_deposit_intro_viewed`: the dot breaks the naming pattern; dual-fire only if a dashboard uses it (src/analytics/event.ts:126)
   - **rename** `cash.deposit_signup_paused_viewed` → `cash_deposit_signup_paused_viewed`: the dot breaks the naming pattern (see the merge item, which would make this rename unnecessary) (src/analytics/event.ts:127)
   - **rename** `cash.existing_account_recovery_selected` → `cash_existing_account_recovery_selected`: the dot breaks the naming pattern (see the merge item, which would make this rename unnecessary) (src/analytics/event.ts:137)
   - **rename** `cash.kyc_approved` → `cash_kyc_approved`: the dot breaks the naming pattern (see the status item) (src/analytics/event.ts:142)
   - **rename** `cash.kyc_awaiting_decision` → `cash_kyc_awaiting_decision`: the dot breaks the naming pattern (see the status item) (src/analytics/event.ts:143)
   - **rename** `cash.kyc_failed` → `cash_kyc_failed`: the dot breaks the naming pattern (see the status item) (src/analytics/event.ts:144)
   - **rename** `cash.kyc_submitted` → `cash_kyc_submitted`: the dot breaks the naming pattern; dual-fire only if a dashboard uses it (src/analytics/event.ts:141)
   - **rename** `cash.lost_passkey_recovery_selected` → `cash_lost_passkey_recovery_selected`: the dot breaks the naming pattern; dual-fire only if a dashboard uses it (src/analytics/event.ts:138)
   - **rename** `cash.passkey_added` → `cash_passkey_added`: the dot breaks the naming pattern; dual-fire only if a dashboard uses it (src/analytics/event.ts:146)
   - **split status** `cash.passkey_failed` → `cash.passkey_added`: rule 4: failure should be status + failure_reason on the outcome event, not a separate _failed event. Add status = success | failed | cancelled and failure_reason (offline | client_error | server_error | unknown). Cancelled, passkey-already-on-device and access-refused currently fire nothing, so add cancelled and a reason for them. (src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:60-73)
   - **split status** `cash.card_link_failed` → `cash.card_linked`: rule 4: card linking failure is a separate _failed event. Fold it into the outcome with status + failure_reason (the draft's card_linked.reason came from this event). Not in this shard's draft list but the code is on the same path. (src/features/cash/stores/cardLinkFlowStore.ts:58)
   - **split status** `cash.kyc_failed` → `cash.kyc_decided`: rule 4: approved, awaiting_decision and failed are three events for one verification outcome. Propose a single outcome event cash.kyc_decided with status = approved | rejected | unsupported_state | awaiting_decision and failure_reason for technical errors (offline | client_error | server_error | unknown), keeping source for the resume/return path. cash.kyc_submitted stays as the intent. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:191,227-235; src/features/cash/services/kycStatusService.ts:21-26)
   - **merge** `cash.kyc_approved` → `cash.kyc_decided`: same outcome as cash.kyc_failed: fold into one outcome event with status (see the split_status item) (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:227)
   - **merge** `cash.kyc_awaiting_decision` → `cash.kyc_decided`: same outcome as cash.kyc_failed: fold in as status = awaiting_decision (see the split_status item), keeping source (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:155)
   - **merge** `cash.deposit_signup_paused_viewed` → `cash.deposit_intro_viewed`: rule 3: one view of the same panel, with the variant set by the cash_signup_enabled flag. Use one event with a property such as signup_enabled = true | false. (src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx:40)
   - **merge** `cash.existing_account_recovery_selected` → `cash.lost_passkey_recovery_selected`: rule 3: two names for one action, choosing account recovery because the passkey is lost. Use one event with entry_point = reauth_sheet | existing_account_prompt, keeping intent where it applies. (useSubmitPhoneFlow.ts:190; ReauthenticateContent.tsx:51)
   - **rename** `cash.passkey_submitted` → `cash_passkey_submitted`: contains a dot, so it breaks the naming pattern ^[a-z][a-z0-9]*(_[a-z0-9]+)+$. Applies to every cash.* event. Deprecate the old name; dual-fire for a window if any dashboard uses it. The name is set in src/analytics/event.ts, so one edit renames all cash.* events. (src/analytics/event.ts:145)
   - **rename** `cash.phone_already_registered` → `cash_phone_already_registered`: dot in the name breaks the naming pattern (src/analytics/event.ts:136)
   - **rename** `cash.phone_resend_failed` → `cash_phone_resend_failed`: dot in the name breaks the naming pattern (src/analytics/event.ts:135)
   - **rename** `cash.phone_submit_failed` → `cash_phone_submit_failed`: dot in the name breaks the naming pattern (only relevant if the event is kept rather than folded into the outcome event, see the split_status item) (src/analytics/event.ts:134)
   - **rename** `cash.phone_submitted` → `cash_phone_submitted`: dot in the name breaks the naming pattern (src/analytics/event.ts:133)
   - **rename** `cash.phone_verified` → `cash_phone_verified`: dot in the name breaks the naming pattern (src/analytics/event.ts:139)
   - **split status** `cash.phone_submit_failed` → `cash.phone_submitted`: failure of the phone submit is a separate _failed event; rule 4 puts it on the outcome event as status = success | failed plus failure_reason. Today every conversion rate needs a union of the two. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78)
   - **split status** `cash.phone_verify_failed` → `cash.phone_verified`: same pattern for verification: the failure is its own event (also fired with the placeholder reason 'client_error' for a rejected code at useSubmitReviewFlow.ts:101, which is a wrong code and not a client error). Fold into the outcome event with status and failure_reason. (src/features/cash/stores/verifyPhoneFlowStore.ts:155)
   - **add pair** `cash.phone_resend_failed`: a failed resend is tracked but a successful resend has no event, so the failure rate cannot be computed. Either add a resend outcome with status success | failed (preferred over a _failed event) or drop the event if nobody asks the question. (src/features/cash/stores/verifyPhoneFlowStore.ts:194)
   - **add pair** `cash.passkey_submitted`: intent with gaps in its outcomes: cash.passkey_added and cash.passkey_failed are only sent for success and generic failure. A cancelled prompt, 'passkey already on this device' and an access refusal send no outcome, so abandonment cannot be told apart from these. Consider status = success | failed | cancelled on one outcome event (cancelled is not a failure). (src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:60-71)
   - **other** `cash.phone_submitted`: outcome and mode values are camelCase ('alreadyRegistered', 'registeredWithPasskey') while the other enum values in this area are snake_case ('client_error', 'server_error'). Consider snake_case values if the event is renamed and dual-fired anyway. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126)
   - **split status** `cash.phone_verify_failed` → `cash_phone_verified`: A _failed event should be status + failure_reason on the outcome event (cash.phone_verified, which also breaks the pattern and needs renaming); keep mode and reason, add status success | failed (src/features/cash/stores/verifyPhoneFlowStore.ts:155)
   - **property** `cash.phone_verify_failed`: Draft lists `source`, which this event never sends; it belongs to cash.kyc_awaiting_decision. Scanner attributed that call to the wrong event (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:155)
   - **split status** `cash.sign_in_failed` → `cash_sign_in_completed`: Failure and cancellation should be status = failed | cancelled with failure_reason on one outcome event paired with sign_in_submitted; cancelled is not a failure (src/features/cash/services/cashSignInService.ts:78)
   - **split status** `cash.sign_in_cancelled` → `cash_sign_in_completed`: Becomes status = cancelled on the single outcome event (src/features/cash/services/cashSignInService.ts:76)
   - **rename** `cash.sign_in_succeeded` → `cash_sign_in_completed`: Becomes the single outcome event with status success | failed | cancelled; dotted name breaks the snake_case pattern. Dual-fire if on a dashboard (src/features/cash/services/cashSignInService.ts:68)
   - **rename** `cash.sign_in_submitted` → `cash_sign_in_submitted`: dotted name breaks the naming pattern (src/features/cash/services/cashSignInService.ts:53)
   - **split status** `cash.wallet_link_failed` → `cash_wallet_linked`: A _failed event should be status + failure_reason on the outcome cash.wallet_linked. Note a cancelled signature stage fires nothing today, so intent (prompted) has no recorded abandon (src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:47)
   - **rename** `cash.wallet_linked` → `cash_wallet_linked`: dotted name breaks the naming pattern; also receives status/failure_reason from the wallet_link_failed merge. Draft's `reason` is never sent on success (src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:36)
   - **rename** `cash.wallet_link_prompted` → `cash_wallet_link_prompted`: dotted name breaks the naming pattern; also fires on every focus, so repeated returns to the sheet are double-counted per attempt (src/features/cash/screens/add-wallet-sheet/AddWalletSheet.tsx:87)
   - **other** `cash.wallet_check_failed`: A _failed event with no outcome event to merge into (the check has no success event) and a dotted name that breaks the pattern. Rename to cash_wallet_check_failed or add a wallet check outcome. Also draft lists amount and entryMode, which belong to cash.amount_entered (src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:495)
   - **remove** `Changed native currency input in Send flow`: Fires per keystroke with no properties; answers no question (rules: nothing per keystroke). Deprecate and remove the call (src/features/transfer/screens/SendSheet.tsx:357)
   - **remove** `Changed token input in Send flow`: Fires per keystroke with no properties; answers no question. Deprecate and remove the call (src/features/transfer/screens/SendSheet.tsx:378)
   - **rename** `Choose Password Step` → `backup_password_step_viewed`: Title-case name breaks the snake_case pattern; keep old name until dashboards move, dual-fire window if on a backup funnel dashboard (src/features/backup/components/BackupCloudStep.tsx:58)
   - **add pair** `Choose Password Step`: Intent with no outcome recorded at this step: no completion event for the cloud backup in this file, so drop-off at the password step cannot be measured unless a backup outcome event exists (src/features/backup/components/BackupCloudStep.tsx:58)
   - **property** `Choose Password Step.category`: category is the constant 'backup' on every call; it carries no information, consider dropping it (src/features/backup/components/BackupCloudStep.tsx:58)
   - **rename** `Clicked "Max" in Send flow input` → `send_max_pressed`: Title-case name with quotes breaks the naming pattern; consider instead a boolean `used_max` on the send outcome event and removing this one (src/features/transfer/components/SendAssetFormField.js:71)
   - **rename** `Completed ENS registration` → `ens_registration_completed`: breaks the snake_case object_action pattern; dual-fire for 30 days if it is on a dashboard. It also fires when the register tx is submitted, not confirmed, so the name overstates what it measures. (src/features/ens/raps/actions.ts:422)
   - **rename** `Initiated ENS registration` → `ens_registration_initiated`: breaks the snake_case pattern; dual-fire for 30 days if it is on a dashboard (src/features/ens/raps/actions.ts:394)
   - **rename** `Edited ENS records` → `ens_records_edited`: breaks the snake_case pattern; dual-fire for 30 days if it is on a dashboard (src/features/ens/raps/actions.ts:400)
   - **rename** `Extended ENS` → `ens_extended`: breaks the snake_case pattern; dual-fire for 30 days if it is on a dashboard (src/features/ens/raps/actions.ts:428)
   - **property** `Edited ENS records.step`: Four different transaction types (multicall, set_text, set_contenthash, set_addr) fire the same event with only a constant category; add a property such as record_action so edits can be told apart (src/features/ens/raps/actions.ts:400, 434, 440, 446)
   - **merge** `Didn't save backup password on iCloud` → `backup_password_saved`: This and 'Saved backup password on iCloud' are the outcome of one action; fold them into one event with status = saved | failed (the catch has no reason, so add failure_reason). Also breaks the naming pattern, so dual-fire for 30 days if it is on a dashboard. (src/features/backup/backup.ts:532-536)
   - **property** `Didn't save backup password on iCloud.category/error/label`: The draft lists category, error and label, but the call at backup.ts:535 sends no properties; they belong to backup.error. Remove them from the draft (or add them to the call). (src/features/backup/backup.ts:187, 535)
   - **rename** `discover.card_pressed` → `discover_card_pressed`: the dot breaks the naming pattern; dual-fire for 30 days if it is on a dashboard (src/features/discover/utils/trackDiscoverCardPress.ts:28)
   - **rename** `discover.carousel_scrolled` → `discover_carousel_scrolled`: the dot breaks the naming pattern (but see the question on whether to keep it) (src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17)
   - **rename** `discover.prediction_order_pressed` → `discover_prediction_order_pressed`: the dot breaks the naming pattern; dual-fire for 30 days if it is on a dashboard (src/features/discover/components/markets/layouts/MarketGrid.tsx:83)
   - **rename** `discover.section_pressed` → `discover_section_pressed`: the dot breaks the naming pattern; dual-fire for 30 days if it is on a dashboard (src/features/discover/components/SectionLayout.tsx:41)
   - **rename** `discover.tab_pressed` → `discover_tab_pressed`: the dot breaks the naming pattern; dual-fire for 30 days if it is on a dashboard (src/features/discover/components/DiscoverHeader.tsx:134)
   - **add pair** `discover.prediction_order_pressed`: This is intent only: whether the order was then placed is not tracked here. Link it to the prediction order outcome event if one exists, or add one. (src/features/discover/components/markets/layouts/MarketCarousel.tsx:87)
   - **rename** `New WalletConnect pairing` → `walletconnect_pairing_received`: breaks the naming pattern; it counts incoming session proposals, not completed pairings. Dual-fire for 30 days if it is on a dashboard. (src/features/wallet-connect/handlers/onSessionProposal.ts:38)
   - **property** `New WalletConnect pairing.type/reason`: The draft lists type and reason, but only the wc.failed_request call (line 175) sends them; the pairing call sends dappName, dappUrl and connector. Remove them from the draft. (src/features/wallet-connect/handlers/onSessionProposal.ts:38, 175)
   - **rename** `Manually disconnected from WalletConnect connection` → `walletconnect_session_disconnected`: breaks the naming pattern; add a property for who ended the session (user) if other disconnects are tracked later. Dual-fire for 30 days if it is on a dashboard. (src/features/wallet-connect/components/WalletConnectV2ListItem.tsx:112)
   - **merge** `notification_permission.granted` → `notification_permission_responded`: Granted and denied are two events for one outcome; fold them into one with status = granted | denied (skipped is a separate intent that stays apart). Do this at the same time as removing the dots from the names, with a 30-day dual-fire if they are on a dashboard. (src/features/notifications/screens/NotificationPermissionScreen.tsx:68-70)
   - **merge** `notification_permission.denied` → `notification_permission_responded`: see the merge on notification_permission.granted; the dot also breaks the naming pattern (src/features/notifications/screens/NotificationPermissionScreen.tsx:70)
   - **rename** `notification_permission.requested` → `notification_permission_requested`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/notifications/screens/NotificationPermissionScreen.tsx:65)
   - **rename** `notification_permission.skipped` → `notification_permission_skipped`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/notifications/screens/NotificationPermissionScreen.tsx:57)
   - **rename** `pair_hw_wallet_nav.entered` → `pair_hw_wallet_nav_entered`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:39)
   - **rename** `pair_hw_wallet_nav.exited` → `pair_hw_wallet_nav_exited`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:42)
   - **rename** `Opened position Sheet` → `position_sheet_opened`: Title case with spaces breaks the naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/positions/components/PositionCard.tsx:78)
   - **rename** `Opened asset from position` → `position_asset_opened`: Title case with spaces breaks the naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/positions/screens/PositionSheet.tsx:81)
   - **rename** `perps.opened_position` → `perps_opened_position`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/perps/components/PerpsNavigatorFooter.tsx:249)
   - **rename** `perps.closed_position` → `perps_closed_position`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/perps/screens/ClosePositionBottomSheet.tsx:150)
   - **rename** `perps.added_to_position` → `perps_added_to_position`: The dot breaks the snake_case naming pattern. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:228)
   - **rename** `perps.trade_details.share_pressed` → `perps_trade_details_share_pressed`: The dots break the snake_case naming pattern. The name also says 'pressed' but the call fires after the share dialog resolves, so consider a name like perps_trade_shared with shared as the outcome. Dual-fire for a window if it is on a dashboard (UNCONFIRMED). (src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:73)
   - **split status** `perps.open_position.failed` → `perps.opened_position`: Failure should be status = success | failed plus failure_reason on the outcome event (rule 4), not a separate _failed event. errorMessage would become failure_reason, with a bounded set of values instead of raw error text. (src/features/perps/components/PerpsNavigatorFooter.tsx:266)
   - **split status** `perps.open_position.canceled` → `perps.opened_position`: A cancel should be status = cancelled on the outcome event, not a separate event. The description should say a cancel is not a failure. (src/features/perps/components/PerpsNavigatorFooter.tsx:237)
   - **split status** `perps.close_position.failed` → `perps.closed_position`: Failure should be status + failure_reason on the outcome event (rule 4), not a separate _failed event. (src/features/perps/screens/ClosePositionBottomSheet.tsx:179)
   - **split status** `perps.added_to_position.failed` → `perps.added_to_position`: Failure should be status + failure_reason on the outcome event (rule 4), not a separate _failed event. (src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:242)
   - **split status** `perps.trigger_order_cancel.failed` → `perps.trigger_order_canceled`: Failure should be status + failure_reason on the sibling success event perps.trigger_order_canceled, not a separate _cancel.failed event. Both calls send the same base properties. (src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71)
   - **other** `perps.open_position.canceled`: A cancelled add-to-position (createIsolatedMarginPosition returned nothing) is reported as perps.open_position.canceled, the same event as a cancelled new position, and carries no marker of which flow it came from. Either add an entry_point/flow property to the cancel, or send it with the add-to-position outcome event. (src/features/perps/screens/perps-add-to-position-sheet/PerpsAddToPositionSheet.tsx:214)
   - **other** `perps.closed_position`: perps.closed_position is tracked before waitForTradeByOrderId and navigation, inside the same try block as the catch that sends perps.close_position.failed. A throw in the later steps makes both events fire for one close. Move the success call to after the last awaited step, or narrow the try. (src/features/perps/screens/ClosePositionBottomSheet.tsx:150)
   - **property** `pair_hw_wallet_nav.entered`: The call sends the route params (entryPoint: AddWalletSheet | ImportOrWatchWalletSheet, flowContext: onboarding | in_app, isFirstWallet: boolean), none of which are in the draft. step is listed but only exited sends it. Add entryPoint, flowContext and isFirstWallet to both pair_hw_wallet_nav events and drop step from entered. (src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:39-45)
   - **property** `Opened position Sheet`: A spread adds positionsUSDValue when the native currency is not USD. It is missing from the draft for position_sheet_opened and the asset event. The asset event also conditionally sends assetValueUSD. Add them as optional properties. (src/features/positions/components/PositionCard.tsx:83)
   - **other** `Viewed external dapp`: The call at PositionSheet.tsx:68 sends the event 'Viewed external dapp' (positionsOpenedExternalDapp), which the scanner attributed to 'Opened asset from position'. That event has no draft entry in this shard. Make sure it gets its own entry. Its url property was drafted onto the asset event by mistake. (src/features/positions/screens/PositionSheet.tsx:68)
   - **property** `Opened asset from position`: assetValue is filled from assetValueUSD, and positionsValue equals positionsUSDValue, but assetCurrency and positionsCurrency hold the native currency. For non-USD users the amount and its currency label disagree. Either convert the amounts or send the currency as USD. (src/features/positions/screens/PositionSheet.tsx:81-93)
   - **property** `Opened asset from position.name`: A property called name matches the never-send pattern ^name$. It is the optional display name of a position item, not personal data, but rename it to position_item_name so it doesn't trip the privacy check. (src/features/positions/screens/PositionSheet.tsx:85)
   - **rename** `perps.trigger_order_canceled` → `perps_trigger_order_canceled`: dotted name breaks the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ pattern; dual-fire for a window if it is on a dashboard (UNCONFIRMED) (src/analytics/event.ts:325)
   - **rename** `perps.trigger_order_created` → `perps_trigger_order_created`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:323)
   - **rename** `perps.withdrew` → `perps_withdrawal`: dotted name breaks the naming pattern, and the past-tense verb differs from predictions.withdrawal; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:327)
   - **rename** `positions.error` → `positions_error`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:176)
   - **rename** `predictions.deposit` → `predictions_deposit`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:345)
   - **rename** `predictions.withdrawal` → `predictions_withdrawal`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:347)
   - **rename** `predictions.place_order` → `predictions_place_order`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:341)
   - **rename** `predictions.redeem_position` → `predictions_redeem_position`: dotted name breaks the naming pattern; dual-fire if on a dashboard (UNCONFIRMED) (src/analytics/event.ts:343)
   - **split status** `perps.trigger_order.failed` → `perps.trigger_order_created`: Rule 4: failure belongs on the outcome event as status plus failure_reason, not a separate _failed event. Keep errorMessage as the failure detail. (src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281)
   - **split status** `perps.withdraw.failed` → `perps.withdrew`: Rule 4: put status = success | failed and failure_reason on the withdrawal outcome event instead of a separate _failed event. (src/features/perps/withdrawalConfig.ts:57)
   - **split status** `predictions.deposit.failed` → `predictions.deposit`: Rule 4: fold into the deposit outcome with status and failure_reason; stage, sponsorshipAttempted and sponsorshipFailureReason become properties of that event. (src/systems/funding/hooks/useDepositHandler.ts:112)
   - **split status** `predictions.withdrawal.failed` → `predictions.withdrawal`: Rule 4: fold into the withdrawal outcome with status and failure_reason; stage and targetChainId stay as properties. (src/systems/funding/hooks/useWithdrawalHandler.ts:128)
   - **split status** `predictions.place_order.failed` → `predictions.place_order`: Rule 4: one order outcome event with status = matched | failed (plus cancelled if the user stops) and failure_reason. place_order currently fires only on match, so it can't serve as a denominator. (src/features/polymarket/screens/polymarket-new-position-sheet/PolymarketNewPositionSheet.tsx:209)
   - **split status** `predictions.order_match.failed` → `predictions.place_order`: Rule 4: this is the second failure branch of the same order outcome (accepted, never matched or settled); fold it into place_order with failure_reason = failed | canceled | expired | rejected | timeout | error. (src/features/polymarket/utils/executePolymarketOrder.ts:353)
   - **split status** `predictions.redeem_position.failed` → `predictions.redeem_position`: Rule 4: put status and failure_reason on the redeem outcome event instead of a separate _failed event. (src/features/polymarket/utils/redeemPosition.ts:82)
   - **other** `perps.trigger_order_cancel.failed`: The code sends this event name for the failed cancel branch, but the draft has no entry for it and the scanner attributed that call to perps.trigger_order_canceled. It needs its own entry (or, per rule 4, a status on the cancel event). (src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:71; src/analytics/event.ts:326)
   - **rename** `Rap started` → `ens_rap_started`: breaks the object_action snake_case pattern (spaces, capital). Keep the old name until the ENS dashboards are moved; dual-fire for 30 days if it is on one. (src/features/ens/raps/actions.ts:717)
   - **rename** `Rap completed` → `ens_rap_completed`: breaks the naming pattern; also see the status item: it currently fires on failure too. (src/features/ens/raps/actions.ts:742)
   - **split status** `Rap failed` → `ens_rap_completed`: Rule 4: failure belongs on the outcome event as status = success | failed plus failure_reason (failed_action). Today Rap completed fires unconditionally after Rap failed, so completed counts include failures. Fix: send ens_rap_completed with status and failed_action, and retire Rap failed. (src/features/ens/raps/actions.ts:689, src/features/ens/raps/actions.ts:742)
   - **rename** `Rejected new WalletConnect session` → `wc_session_rejected`: breaks the naming pattern (spaces, capitals). Same family as the other wc.* / 'Approved new WalletConnect session' names, which should be renamed together. (src/features/wallet-connect/handlers/onSessionProposal.ts:62)
   - **property** `Rejected new WalletConnect session.connector, .type, .reason`: The draft lists connector, type and reason, but the only call that sends this event (line 62) sends dappName and dappUrl. connector comes from the wcNewPairing call (line 38) and type/reason from the wc.failed_request call (line 175). Remove them from this entry; the scanner mixed calls from the same file. (src/features/wallet-connect/handlers/onSessionProposal.ts:38)
   - **property** `Rejected new WalletConnect session`: Add a reason property (e.g. the getSdkError reason passed to rejectProposal) so rejections can be broken down. The value is already available in rejectProposal. (src/features/wallet-connect/handlers/onSessionProposal.ts:45)
   - **rename** `request.approved` → `dapp_request_approved`: Dot-separated name breaks the pattern. The name 'request' alone is too generic across products; dual-fire for 30 days if on a dashboard. (src/features/dapp-request/screens/SignTransactionSheet.tsx:406)
   - **rename** `request.rejected` → `dapp_request_rejected`: Dot-separated name breaks the naming pattern. (src/features/dapp-request/screens/SignTransactionSheet.tsx:295)
   - **rename** `request.sheet.show` → `dapp_request_sheet_shown`: Dot-separated name breaks the naming pattern. (src/features/dapp-request/hooks/useTransactionSetup.ts:55)
   - **other** `request.sheet.show`: Effect dependencies (chainId, request params, source, gas callbacks) can re-run the effect, so the event may fire more than once per sheet. Guard it to once per sheet open, or the funnel to approved/rejected is overstated. (src/features/dapp-request/hooks/useTransactionSetup.ts:46)
   - **property** `request.rejected`: request.rejected sends only source, requestType and isHardwareWallet, whereas request.approved also sends dappName, dappUrl and network. Add the three to the rejected call so approve vs reject can be broken down by dapp and network. The draft marks them as optional because of the shared draft. (src/features/dapp-request/screens/SignTransactionSheet.tsx:295)
   - **split status** `request.rejected` → `dapp_request_approved`: request.rejected is also sent when message signing returns an error (onCancel(response.error)), so signing failures are counted as user rejections. Rule 4: put status = success | cancelled | failed with failure_reason on one outcome event (cancelled is not a failure). (src/features/dapp-request/screens/SignTransactionSheet.tsx:523)
   - **property** `Reset Asset Selection Send`: The draft lists screen, no_icon, no_price, total_tokens, but those are sent by the token_list event (SendSheet.tsx:670); Reset Asset Selection Send is sent with no properties at line 554. The scanner also lists lines 357, 378 which send other events (changedNativeCurrencyInputSend, changedTokenInputSend). Remove these properties from the entry. (src/features/transfer/screens/SendSheet.tsx:554)
   - **remove** `Reset Asset Selection Send`: A UI reset of a selection in an input form: no question a product owner asks on a Monday. Candidate to remove; the same applies to the per-input events changed_token_input_send and changed_native_currency_input_send (other shards). (src/features/transfer/screens/SendSheet.tsx:553)
   - **rename** `rnbw_airdrop.claim` → `rnbw_airdrop_claim`: Dot in the name breaks the object_action pattern; dual-fire for 30 days if on a dashboard. (src/features/rnbw-rewards/utils/claimAirdrop.ts:91)
   - **split status** `rnbw_airdrop.claim.failed` → `rnbw_airdrop_claim`: Rule 4: failures belong on the outcome event as status = success | failed plus failure_reason (today errorMessage, which is free-form exception text), not on a separate .failed event. Funnels would otherwise drop the failures. (src/features/rnbw-rewards/utils/claimAirdrop.ts:116)
   - **rename** `rnbw_airdrop.claim.failed` → `rnbw_airdrop_claim_failed`: Dot in the name breaks the naming pattern; only needed for the transition window before the merge into rnbw_airdrop_claim. (src/features/rnbw-rewards/utils/claimAirdrop.ts:49)
   - **rename** `rnbw_rewards.claim` → `rnbw_rewards_claim`: Dot in the name breaks the naming pattern; dual-fire for 30 days if on a dashboard. (src/features/rnbw-rewards/utils/claimRewards.ts:123)
   - **split status** `rnbw_rewards.claim.failed` → `rnbw_rewards_claim`: Rule 4: failure belongs on the outcome event as status and failure_reason, not a separate .failed event. The code calls rnbw_rewards.claim.failed at lines 75 and 151, but that event isn't drafted in this shard, so it's raised here once. (src/features/rnbw-rewards/utils/claimRewards.ts:151)
   - **property** `rnbw_rewards.claim`: The draft carries errorMessage, but only the failed calls (lines 75, 151) send it; the success call (line 123) doesn't. Remove it from the success entry, as for the airdrop claim. (src/features/rnbw-rewards/utils/claimRewards.ts:123)
   - **add pair** `rnbw_airdrop.claim`: Rule 2: a claim can be abandoned (user rejects signing, leaves the sheet) and abandonment is only visible if there is an intent event, e.g. rnbw_airdrop_claim_started on the claim button. Note that a user cancelling signature is currently recorded as a failure. (src/features/rnbw-rewards/utils/claimAirdrop.ts:28)
   - **add pair** `rnbw_rewards.claim`: Rule 2: same as the airdrop claim; no intent event before GetClaimIntent / signing, so drop-off before the outcome is not measurable. (src/features/rnbw-rewards/utils/claimRewards.ts:51)
   - **split status** `rnbw_rewards.claim.failed` → `rnbw_rewards_claimed`: Failure lives on the outcome as status + failure_reason, not a separate _failed event (rule 4). Note rnbw_rewards.claim already sends a backend `status` string, so use a distinct property name for the outcome or confirm what that field holds. The name also contains dots and breaks the naming pattern, so rename the success event to rnbw_rewards_claimed. Dual-fire if it is on a dashboard. (src/features/rnbw-rewards/utils/claimRewards.ts:75,151)
   - **rename** `rnbw_staking.stake` → `rnbw_staking_staked`: Dots break the naming pattern; deprecate the old name and dual-fire if it is on a dashboard. (src/features/rnbw-staking/utils/stakeRnbw.ts:194)
   - **split status** `rnbw_staking.stake.failed` → `rnbw_staking_staked`: Fold the failure into the outcome event as status + failure_reason (rule 4). Add a failure_reason that separates failure before submit (line 157) from failure after submit (line 182). (src/features/rnbw-staking/utils/stakeRnbw.ts:157,182,213)
   - **rename** `rnbw_staking.unstake` → `rnbw_staking_unstaked`: Dots break the naming pattern; deprecate the old name and dual-fire if it is on a dashboard. (src/features/rnbw-staking/utils/unstakeRnbw.ts:111)
   - **split status** `rnbw_staking.unstake.failed` → `rnbw_staking_unstaked`: Fold the failure into the outcome event as status + failure_reason (rule 4). Add a failure_reason that separates failure before submit (line 73) from failure after submit (line 119). (src/features/rnbw-staking/utils/unstakeRnbw.ts:73,119)
   - **rename** `spot_pnl.sheet.viewed` → `spot_pnl_sheet_viewed`: Dots break the naming pattern; deprecate the old name and dual-fire if it is on a dashboard. (src/features/spot-pnl/ui/SpotPnlSheet.tsx:257)
   - **rename** `surface.interaction` → `surface_interacted`: Dots break the naming pattern; deprecate the old name and dual-fire if it is on a dashboard. (src/features/placements/engagement/trackInteraction.ts:43)
   - **rename** `Saved backup password on iCloud` → `backup_password_saved`: Title-case sentence breaks the naming pattern. Also merge with the sibling "Didn't save backup password on iCloud" (backup.ts:535) into one event with status success | failed (rule 4). Deprecate the old names and dual-fire if either is on a dashboard. (src/features/backup/backup.ts:532,535)
   - **rename** `Sent transaction` → `transaction_sent`: Title-case name breaks the naming pattern. It already carries submitSuccessful, which should become status (success | failed) with a failure_reason. (src/features/transfer/utils/trackSentTransaction.ts:32)
   - **rename** `Set App Icon` → `app_icon_set`: Title-case name breaks the naming pattern. Also consider removing it (see questions). (src/features/app-icon/components/AppIconSettingsSection.tsx:32)
   - **rename** `Showing Walletconnect signing request` → `wallet_connect_signing_request_shown`: Title-case name breaks the naming pattern. The failure side is a separate wc.failed_request event, which also breaks the pattern and should become status + failure_reason on a request outcome event. There is no outcome event (approved or rejected) in the code I saw. (src/features/wallet-connect/handlers/onSessionRequest.ts:204)
   - **rename** `Tapped "Confirm Backup"` → `backup_confirmed`: Title-case name with embedded quotes breaks the naming pattern. It is the intent half of a pair (outcomes: backupComplete, backupError), so keep it and consider folding the outcomes into one backup_completed with status. (src/features/backup/hooks/useCreateBackup.ts:92)
   - **rename** `Transferred ENS control` → `ens_control_transferred`: Title-case name breaks the naming pattern. It fires after the tx is sent, not confirmed, so say so in the description. The ENS actions (ensInitiatedRegistration, ensEditedRecords, ensExtended, ensTransferredControl, ensSetPrimary) are separate events with one `category` property, a candidate for a single ens_action event with an `action` property (rule 3). (src/features/ens/raps/actions.ts:394,400,422,428,452,458)
   - **rename** `Updated Gas Price` → `gas_price_updated`: Title-case name breaks the naming pattern. The event is named for swaps but is fired from the shared gas action, so the name is misleading. (src/features/gas/redux/gas.ts:488)
   - **rename** `Viewed external dapp` → `position_dapp_opened`: The name says 'viewed' but the event fires when the user taps to open the dapp website. Title-case breaks the naming pattern. (src/features/positions/screens/PositionSheet.tsx:68)
   - **other** `rnbw_rewards.claim.failed`: The draft's properties and call sites were pooled from rnbw_rewards.claim (line 123), which is the success event. claimedRnbw, claimedValueInCurrency, decimals, txHash, tenderlyUrl and pollAttempts are never sent by a failure call. Remove them from this entry. The scanner should attribute the call at line 123 to rnbw_rewards.claim, not to the failed event. (src/features/rnbw-rewards/utils/claimRewards.ts:123)
   - **property** `rnbw_staking.stake`: The success call spreads StakeSuccessMetadata, which also sends amountFromWallet and amountFromRewards. The draft lacks both. The draft's errorMessage is sent only by the failure event. (src/features/rnbw-staking/utils/stakeRnbw.ts:33-40,194)
   - **property** `rnbw_staking.unstake`: The success call spreads analyticsSnapshot, which sends expectedExitFee, expectedReceiveAmount and pnl in addition to stakedAmount (receiveRaw is also in the snapshot and appears to be sent). The draft lacks these. The draft's errorMessage is sent only by the failure event. (src/features/rnbw-staking/utils/unstakeRnbw.ts:111-117,129-139)
   - **property** `spot_pnl.sheet.viewed`: The call spreads viewSummary, which sends variant, unrealizedPnlUsd, realizedPnlUsd and lastTradeAt per the event type. The draft lacks all four. (src/features/spot-pnl/ui/SpotPnlSheet.tsx:257-266)
   - **property** `Viewed external dapp`: The call conditionally adds positionsUSDValue when the currency is not USD. The draft lacks it. The asset* and type properties in the draft belong to positionsOpenedAsset (line 81), not this event. (src/features/positions/screens/PositionSheet.tsx:68-75,81-93)
   - **other** `surface.interaction`: The draft pooled placement.interaction (trackPlacementInteraction, line 20) into this entry. interactionType, itemId, itemOrder, source, surfaceId and type are not sent by surface.interaction. Placement interactions need their own entry. (src/features/placements/engagement/trackInteraction.ts:20,43)
   - **other** `Saved backup password on iCloud`: The draft pooled backupError (line 187) into this entry, giving it category, error and label. They are not sent by this event. The sibling calls at lines 532 and 535 are two different events (saved vs. skipped) and the scanner treated them as one. (src/features/backup/backup.ts:187,532,535)
   - **other** `Tapped "Confirm Backup"`: category and label come from backupComplete (line 65), not from this event. This event sends no properties of its own. (src/features/backup/hooks/useCreateBackup.ts:65,92)
   - **other** `Transferred ENS control`: failed_action and label come from other ENS rap events (ensRapFailed, ensRapStarted, ensRapCompleted). The other ENS actions at lines 394, 400 and 422 are also different events pooled into this entry. (src/features/ens/raps/actions.ts:394,422,689,717)
   - **other** `Showing Walletconnect signing request`: Lines 84, 119 and 147 are wc.failed_request calls. Their type, reason and method properties are not sent by this event. (src/features/wallet-connect/handlers/onSessionRequest.ts:84,119,147,214)
   - **privacy** `Viewed external dapp.name`: `name` is a banned never-send name pattern (^name$). It is the optional display name sent by 'Opened asset from position' and pooled into this draft. Rename it to position_name or drop it. (src/features/positions/screens/PositionSheet.tsx:85)
   - **property** `Sent transaction.isRecepientENS`: Misspelled property name (Recepient). Add isRecipientENS alongside it and dual-send, then deprecate the typo. (src/features/transfer/utils/trackSentTransaction.ts:40)
   - **rename** `wc.failed_request` → `wc_request_failed`: The name contains a dot and breaks the snake_case naming pattern. Keep the old name and dual-fire for a window if it is on a dashboard. Also consider whether it belongs on a WalletConnect outcome event with status + failure_reason (rule 4): these failures are mixed refusals (no request formed) and request failures, so decide with the owner. (src/analytics/event.ts:171)
   - **property** `Manually disconnected from WalletConnect connection`: Disconnect from a dapp on `Routes.DAPP_BROWSER_CONTROL_PANEL` (secondary) has no event: It is the same action as the WalletConnect disconnect, so one event with a source property separates the two surfaces. It fires in handleDisconnect after removeAppSession. Existing rows get no source value. Add `source`, `dappUrl`. (src/features/dapp-browser/screens/ControlPanel.tsx#ControlPanel)
   - **add event** `backup_restore_completed`: Restore wallet from cloud backup on `Routes.BACKUP_SHEET`; Enter backup password on `Routes.RESTORE_CLOUD_SHEET` (core) has no event. Answers how many restore attempts succeed and why they fail. Nothing records this today, only backup creation. It fires in RestoreCloudStep.onSubmit from the status of restoreCloudBackup, and from the catch branch for errors. The intent event backup_restore_started is also worth adding when the password is submitted, so drop-off before submit is measurable. Properties: `status`, `failure_reason`, `cloud_platform`, `entry_point`, `is_first_wallet`. (src/features/backup/components/BackupSheet.tsx#BackupSheet, src/features/backup/components/RestoreCloudStep.tsx#RestoreCloudStep)
   - **add event** `cash_deposit_intro_cta_pressed`: Set up cash account on `Routes.CASH_DEPOSIT_INTRO_PANEL` (core) has no event. The intro view event and the phone submit event have nothing between them. This measures how many people who see the intro start setup, and how many are refused because the wallet is a hardware wallet. handleSetUpAccount shows an alert for hardware wallets before it navigates, which nothing else records. The same event also carries the sign-in and other-deposit-methods presses through the cta property. Properties: `cta`, `signup_enabled`, `blocked_reason`. (src/features/cash/screens/cash-deposit-intro-panel/CashDepositIntroPanel.tsx#CashDepositIntroPanel)
   - **add event** `hardware_wallet_pair_completed`: Search for and select a Ledger device on `Routes.PAIR_HARDWARE_WALLET_NAVIGATOR` (core) has no event. pair_hw_wallet_nav.entered and .exited show who starts and who leaves, but not whether a device was found and paired or what blocked it. It fires from useLedgerImport when the device pairs or the search errors. The intent is the existing nav entered event. Properties: `status`, `failure_reason`, `flow_context`. (src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx#PairHardwareWalletNavigator)
   - **add event** `send_asset_selected`: Enter a recipient and select an asset to send on `Routes.SEND_SHEET` (core) has no event. Sent transaction is the only outcome, and there is no event when the person picks an asset after entering a recipient. This lets you measure how many reach the amount step but never send. Entering a recipient is not a separate event because it only changes the form state. Properties: `asset_type`, `chain_id`, `recipient_type`, `is_sponsored_send`. (src/features/transfer/screens/SendSheet.tsx#SendSheet)
   - **add event** `cash_card_removed`: Remove linked card on `Routes.CASH_PAYMENT_METHODS_SHEET` (secondary) has no event. The plan has card linking events but nothing for removing the card. Churn from Cash and failed removals depend on it. It fires after cardRemovalFlowActions.remove resolves, from the confirm button. 'Keep card' is the abandon branch and is not worth its own event. Properties: `status`, `brand`. (src/features/cash/screens/payment-methods-sheet/PaymentMethodsSheet.tsx#PaymentMethodsSheet)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
2. **screens** (one PR)
   - **rename** `add_wallet_flow.started` → `add_wallet_flow_started`: The dot breaks the naming pattern ^[a-z][a-z0-9]*(_[a-z0-9]+)+$. Check dashboards for the old name first and dual-fire if any use it. (src/screens/AddWalletSheet.tsx:45)
   - **rename** `analytics_tracking.disabled` → `analytics_tracking_disabled`: The dot breaks the naming pattern. (src/screens/SettingsSheet/components/PrivacySection.tsx:35)
   - **rename** `analytics_tracking.enabled` → `analytics_tracking_enabled`: The dot breaks the naming pattern. (src/screens/SettingsSheet/components/PrivacySection.tsx:42)
   - **rename** `Changed language` → `language_changed`: Title-case with spaces breaks the snake_case object_action pattern. Dual-fire if it is on a dashboard. (src/screens/SettingsSheet/components/LanguageSection.tsx:24)
   - **rename** `Changed native currency` → `native_currency_changed`: Title-case with spaces breaks the naming pattern. (src/screens/SettingsSheet/components/CurrencySection.tsx:40)
   - **rename** `Changed network` → `network_changed`: Title-case with spaces breaks the naming pattern. (src/screens/SettingsSheet/components/NetworkSection.tsx:30)
   - **split status** `claim_claimable.failed` → `claim_claimable_completed`: Rule 4: success and failure should be one outcome event with status = success | failed | cancelled and failure_reason, not claim_claimable.succeeded plus claim_claimable.failed. Every claim conversion rate now needs a union. Also, the dot breaks the naming pattern. (src/screens/Airdrops/utils.ts:121)
   - **merge** `claim_claimable.succeeded` → `claim_claimable_completed`: Folds into the single outcome event with status (see claim_claimable.failed). The dot also breaks the naming pattern. (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:136)
   - **add pair** `claim_claimable.succeeded`: No intent event exists for a claim (button press). A claim with no outcome (for example sponsored biometrics failure, which returns without tracking) is invisible. Consider a claim_claimable_started event. (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:57)
   - **other** `claim_claimable.failed`: A user rejecting or cancelling the transaction is sent as a failure (isUserCancelled is computed for the log but not sent). It should be status = cancelled. (src/screens/Airdrops/utils.ts:140)
   - **rename** `f2c.provider.flow_opened` → `f2c_provider_flow_opened`: The dots break the naming pattern. The code key is f2cProviderFlowStarted but the sent name is flow_opened, so the name and the code key disagree. Sibling events f2c.provider.flow_completed and f2c.provider.flow_errored have the same naming problem and use the `.provider.` form. (src/screens/FiatOnRamp/ProviderListItem.tsx:35)
   - **other** `f2c.provider.flow_opened`: Intent with outcome through an external callback (the Moonpay redirect only). For the other providers (ramp, coinbase, stripe) no redirect URI is passed, so no outcome can be joined to the session. Abandonment cannot be measured for them. (src/screens/FiatOnRamp/ProviderListItem.tsx:30)
   - **rename** `learn_article.opened` → `learn_article_viewed`: The dot breaks the naming pattern, and the event fires when the screen closes, not when the article opens, so 'opened' is misleading. 'viewed' with durationSeconds says what it measures. (src/screens/LearnWebViewScreen.tsx:35)
   - **rename** `learn_article.shared` → `learn_article_shared`: The dot breaks the naming pattern. (src/screens/LearnWebViewScreen.tsx:51)
   - **rename** `Received wc connection` → `wc_connection_received`: Title-case with spaces breaks the naming pattern. Dual-fire if it is on a dashboard. (src/screens/WalletConnectApprovalSheet.tsx:298)
   - **property** `Received wc connection.waitingTime`: The same property is either a number of seconds or the string 'Error calculating waiting time.'. Send a number or leave it out. (src/screens/WalletConnectApprovalSheet.tsx:301)
   - **add pair** `Received wc connection`: A connection request can be approved, rejected or time out, but no outcome event is sent from this sheet (handleConnect and handleCancel only call the callback). Consider adding a wc_connection_completed event with status = approved | rejected | timed_out. (src/screens/WalletConnectApprovalSheet.tsx:271)
   - **add pair** `add_wallet_flow.started`: There is no outcome event for adding a wallet (wallet created, imported or watched), so drop-off between starting and finishing cannot be measured from this entry. Check whether one already exists elsewhere before adding it. (src/screens/AddWalletSheet.tsx:45)
   - **other** `add_wallet_flow.started`: The scanner also found a call in ViewWalletBackup.onCreateNewWallet that its list for this shard missed (isFirstWallet false, type new). It has to be included in the code refs. Separate events (tappedCreateNewWallet, tappedAddExistingWallet, tappedWatchAddress) fire on the same taps in AddWalletSheet and duplicate this intent. (src/screens/SettingsSheet/components/Backups/ViewWalletBackup.tsx:140)
   - **merge** `rewards.pressed_available_card` → `rewards_card_pressed`: Rewards card presses (available, bridged, swapped, pending earnings, position, leaderboard item) are one action from different places: one event with a card property (available | bridged | swapped | pending_earnings | position | leaderboard_item). Keep the old names until dashboards move, dual-fire window if they're on one. Also breaks the naming pattern (dots). (src/screens/rewards/components/RewardsClaimed.tsx:87)
   - **merge** `rewards.pressed_bridged_card` → `rewards_card_pressed`: Same merge as rewards.pressed_available_card: card = bridged. (src/screens/rewards/components/RewardsStats.tsx:34)
   - **merge** `rewards.pressed_swapped_card` → `rewards_card_pressed`: Same merge as rewards.pressed_available_card: card = swapped. (src/screens/rewards/components/RewardsStats.tsx:42)
   - **merge** `rewards.pressed_pending_earnings_card` → `rewards_card_pressed`: Same merge as rewards.pressed_available_card: card = pending_earnings. (src/screens/rewards/components/RewardsEarnings.tsx:71)
   - **rename** `rewards.pressed_leaderboard_item` → `rewards_leaderboard_item_pressed`: Breaks the object_action snake_case pattern (dot separator). Alternatively fold into rewards_card_pressed with card = leaderboard_item. (src/screens/rewards/components/RewardsLeaderboardItem.tsx:81)
   - **rename** `rewards.viewed_sheet` → `rewards_sheet_viewed`: Breaks the naming pattern (dot). It also fires on every focus, not only the first open, so returning from a pushed screen counts again; consider firing on mount. (src/screens/rewards/RewardsSheet.tsx:45)
   - **rename** `Show Secret View` → `secret_view_shown`: Title case with spaces breaks the snake_case pattern. 'Secret' is not a banned word in the event name, and the event sends no secret. (src/screens/SettingsSheet/components/Backups/ShowSecretView.tsx:10)
   - **rename** `Shown Walletconnect session request` → `walletconnect_session_request_shown`: Breaks the naming pattern (capitals, spaces). (src/screens/WalletConnectApprovalSheet.tsx:260)
   - **property** `Shown Walletconnect session request`: The draft lists dappName, dappUrl and waitingTime, but the call at line 260 sends none of them. They are sent by the separate 'Received wc connection' event at line 298, which the scanner attributed to this one. Remove the three from this entry and draft them on 'Received wc connection'. (src/screens/WalletConnectApprovalSheet.tsx:298)
   - **add pair** `Shown Walletconnect session request`: Intent with no outcome: approve (handleConnect), cancel (handleCancel) and dismissal (counted as rejection) call the callback but are not tracked. Add an outcome event, e.g. walletconnect_session_request_resolved with status = success | cancelled | failed. (src/screens/WalletConnectApprovalSheet.tsx:271)
   - **merge** `Received wc connection`: Fires on the same sheet mount as 'Shown Walletconnect session request' (two effects, both after interactions) with dapp name, url and waiting time. Two events for one moment: consider merging them, with the dapp properties on one. Not in this shard's drafts, so confirm and draft it. (src/screens/WalletConnectApprovalSheet.tsx:298)
   - **rename** `spot_pnl.pill.shown` → `spot_pnl_pill_shown`: Breaks the naming pattern (dots). Event type declares chainId as number, but the draft typed it string; check which the data holds. (src/screens/expandedAssetSheet/components/sections/BalanceSection.tsx:61)
   - **rename** `Tapped "Cancel" after editing wallet` → `wallet_edit_closed`: Breaks the naming pattern, and the name is wrong: it fires on the modal closing with no name, not on a Cancel tap. Better: one wallet_edit_completed event with status = saved | cancelled (cancelled is not a failure), absorbing the Done event. (src/screens/change-wallet/ChangeWalletSheet.tsx:267)
   - **merge** `Tapped "Done" after editing wallet` → `wallet_edit_completed`: Done and Cancel are the two outcomes of one edit-wallet flow; fold into one event with status = saved | cancelled. Also breaks the naming pattern. (src/screens/change-wallet/ChangeWalletSheet.tsx:264)
   - **privacy** `Tapped "Done" after editing wallet.wallet_label`: wallet_label is the raw name the person typed for the wallet (free text). Never send raw user text; send label_char_count or a has_label boolean instead. (src/screens/change-wallet/ChangeWalletSheet.tsx:264)
   - **rename** `Tapped "Edit Wallet"` → `wallet_edit_started`: Breaks the naming pattern (capitals, quotes, spaces). It is the intent of the wallet edit flow, paired with the completion event. (src/screens/change-wallet/ChangeWalletSheet.tsx:286)
   - **add pair** `Tapped "Delete Wallet" (final confirm)`: Confirmation is logged before deleteWallet runs; there is no outcome event for deletion success or failure. Add wallet_deleted (server-independent, client) with status = success | failed, and make the confirm tap the intent. Also pair with 'Tapped Delete Wallet' (first tap) as the earlier step. (src/screens/change-wallet/ChangeWalletSheet.tsx:341)
   - **rename** `Tapped "Delete Wallet" (final confirm)` → `wallet_delete_confirmed`: Breaks the naming pattern (capitals, quotes, parentheses). (src/screens/change-wallet/ChangeWalletSheet.tsx:341)
   - **other** `Tapped "Notification Settings"`: The scanner mapped line 295 (tappedNotificationSettings) onto the wallet events by mistake; the event has no draft in this shard. Mention so it isn't missed: it is a separate event and needs its own entry. (src/screens/change-wallet/ChangeWalletSheet.tsx:295)
   - **rename** `Tapped "Get a new wallet"` → `onboarding_wallet_create_started`: Breaks the naming pattern (capitals, quotes, spaces). (src/screens/WelcomeScreen/WelcomeScreen.tsx:163)
   - **add pair** `Tapped "Get a new wallet"`: Intent with no outcome: wallet creation can fail (RainbowError, alert 'Error creating wallet') and no event records success or failure. Add wallet_created with status = success | failed and failure_reason. (src/screens/WelcomeScreen/WelcomeScreen.tsx:173)
   - **merge** `Tapped "I already have one"` → `onboarding_wallet_create_started`: Both buttons are entry choices into wallet setup from one screen. Consider one event onboarding_path_selected with path = new_wallet | existing_wallet, instead of two. Also breaks the naming pattern (capitals, quotes). (src/screens/WelcomeScreen/WelcomeScreen.tsx:201)
   - **rename** `Tapped "Notification Settings"` → `wallet_notification_settings_tapped`: Quotes and spaces break the object_action snake_case pattern. Keep the old name and dual-fire for 30 days if it is on a dashboard (not known). (src/screens/change-wallet/ChangeWalletSheet.tsx:295)
   - **property** `Tapped "Notification Settings"`: The draft lists action, buttonName and wallet_label, but the call sends no properties. The scanner merged keys from neighbouring calls in the file. Drop them from the entry; only walletAddressHash and walletType (added by analytics.track) arrive. (src/screens/change-wallet/ChangeWalletSheet.tsx:295)
   - **rename** `Tapped Delete Wallet` → `wallet_delete_tapped`: Breaks the snake_case pattern. Dual-fire for 30 days if it is on a dashboard. (src/screens/change-wallet/ChangeWalletSheet.tsx:317)
   - **property** `Tapped Delete Wallet`: The draft lists action, buttonName and wallet_label, but the call sends no properties. These were merged in from neighbouring calls. Drop them from the entry. (src/screens/change-wallet/ChangeWalletSheet.tsx:317)
   - **add pair** `Tapped Delete Wallet`: Intent without a real outcome. The only follow-up is a tap on the confirm button (Tapped "Delete Wallet" (final confirm)), which fires before deleteWallet runs. Add a wallet_deleted outcome after deleteWallet resolves, with status success | failed. (src/screens/change-wallet/ChangeWalletSheet.tsx:341)
   - **remove** `Tapped Edit`: It only toggles a UI edit mode, fires on both entering and leaving it, and answers no product question. Edits that matter are captured by the "Done" after editing wallet event. Propose deprecating it. (src/screens/change-wallet/ChangeWalletSheet.tsx:411)
   - **property** `Tapped Edit`: The draft lists action, buttonName and wallet_label, but the call sends no properties. Drop them from the entry. (src/screens/change-wallet/ChangeWalletSheet.tsx:411)
   - **merge** `Tapped Add Existing Wallet` → `add_wallet_flow.started`: It fires in the same handler as add_wallet_flow.started with type 'seed', so it is a second name for one action (rule 3). Remove the call and use that event's type value. add_wallet_flow.started also breaks the naming pattern (the dot) and should be renamed to add_wallet_flow_started. (src/screens/AddWalletSheet.tsx:58-59)
   - **merge** `Tapped Create New Wallet` → `add_wallet_flow.started`: It fires in the same handler as add_wallet_flow.started with type 'new', so it duplicates it. Remove the call and use that event's type value. (src/screens/AddWalletSheet.tsx:45-49)
   - **merge** `Tapped Watch Address` → `add_wallet_flow.started`: It fires in the same handler as add_wallet_flow.started with type 'watch', so it duplicates it. Remove the call and use that event's type value. (src/screens/AddWalletSheet.tsx:70-72)
   - **property** `Tapped Add Existing Wallet`: The draft lists isFirstWallet and type, but this call sends neither. They are sent on the add_wallet_flow.started call next to it. isFirstWallet is a boolean in code, not a string. Drop both from this entry. (src/screens/AddWalletSheet.tsx:58-62)
   - **property** `Tapped Create New Wallet`: The draft lists isFirstWallet and type, but this call sends neither. They belong to add_wallet_flow.started. (src/screens/AddWalletSheet.tsx:45-49)
   - **property** `Tapped Watch Address`: The draft lists isFirstWallet and type, but this call sends neither. They belong to add_wallet_flow.started. (src/screens/AddWalletSheet.tsx:70-74)
   - **rename** `token_launcher.abandoned` → `token_launcher_abandoned`: The dot breaks the snake_case pattern. Dual-fire for 30 days if it is on a dashboard. (src/screens/token-launcher/components/TokenLauncherHeader.tsx:59)
   - **property** `token_launcher.abandoned`: The call spreads getAnalyticsParams(), so it also sends chainId, symbol, name, logoUrl, description, totalSupply, links, extraBuyAmount and imageModerated. The draft lists only the wallet context. Add the ones that are kept after the privacy fix. (src/screens/token-launcher/components/TokenLauncherHeader.tsx:59-61)
   - **rename** `token_launcher.share_pressed` → `token_launcher_share_pressed`: The dot breaks the snake_case pattern. It also fires after the share sheet closes, not on press, so the description says so. Dual-fire for 30 days if it is on a dashboard. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:212)
   - **property** `token_launcher.share_pressed`: The draft has error, but this call never sends it. The scanner merged it in from the tokenLauncherWalletLoadFailed calls in the same file. The call also spreads getAnalyticsParams(), which adds chainId, symbol, name, logoUrl, description, totalSupply, links, extraBuyAmount and imageModerated; the draft lacks these. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:212-216)
   - **other** `token_launcher.wallet_load_failed`: The scanner attributed its three call sites (TokenLauncherFooter.tsx:61, 79, 120) to token_launcher.share_pressed. The event is missing from the entries given here, so check that it has its own entry. As a _failed event it should probably be folded into token_launcher_token_created as status and failure_reason. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:61)
   - **rename** `token_launcher.step_changed` → `token_launcher_step_changed`: The dot breaks the snake_case pattern. Dual-fire for 30 days if it is on a dashboard. (src/screens/token-launcher/state/tokenLauncherStore.ts:292)
   - **property** `token_launcher.step_changed`: The call sends only step. The draft also lists address, error, operation, source and transactionHash, which come from the token_created and creation_failed calls in the same file. The scanner reports lines 292, 379 and 438 against all three events. Drop them from this entry. (src/screens/token-launcher/state/tokenLauncherStore.ts:292-294)
   - **rename** `token_launcher.token_created` → `token_launcher_token_created`: The dot breaks the snake_case pattern. It should also carry status = success | failed and failure_reason (see the creation_failed merge). Dual-fire for 30 days if it is on a dashboard. (src/screens/token-launcher/state/tokenLauncherStore.ts:379)
   - **property** `token_launcher.token_created`: The call sends address plus the getAnalyticsParams() spread (chainId, symbol, name, logoUrl, description, totalSupply, links, extraBuyAmount, imageModerated). The draft also lists step, error, operation, source and transactionHash, which this call does not send, and lacks the spread properties. (src/screens/token-launcher/state/tokenLauncherStore.ts:379-382)
   - **merge** `token_launcher.creation_failed` → `token_launcher.token_created`: A failure belongs on the outcome event as status = success | failed with failure_reason (rule 4), not on a separate _failed event. Add status to token_created, and carry error, operation, source and transactionHash on the failed case. (src/screens/token-launcher/state/tokenLauncherStore.ts:438)
   - **property** `token_launcher.creation_failed`: The call sends the getAnalyticsParams() spread plus error, operation, source and transactionHash. The draft lacks the spread properties and lists step (not sent here) and address (typed but never set). (src/screens/token-launcher/state/tokenLauncherStore.ts:438-444)
   - **other** `token_launcher.creation_failed`: The try block in createToken also wraps getTransaction and addNewTransaction, which run after token_created fires. A failure there fires creation_failed for a token that was already created, so the same launch is counted as both success and failure. Narrow the try, or add a stage property. (src/screens/token-launcher/state/tokenLauncherStore.ts:360-446)
   - **split status** `token_launcher.image_upload_failed` → `token_launcher_image_uploaded`: There is no success event for the image upload, so the failure rate can't be computed. Add an outcome event with status = success | failed and failure_reason, and fold this event into it. The current name also breaks the snake_case pattern. (src/screens/token-launcher/hooks/useUploadToCloudinary.ts:17-33)
   - **property** `token_launcher.image_upload_failed`: url and isModerated are assigned only after the success check and the return, so in the catch block they are always undefined. They never arrive. Either remove them or send values from the failed response. isModerated is a boolean, not a string. (src/screens/token-launcher/hooks/useUploadToCloudinary.ts:14-33)
   - **rename** `token_launcher.wallet_load_failed` → `token_launcher_wallet_load_failed`: A dot in the name breaks the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ pattern. Keep the old name until dashboards are checked, then dual-fire for 30 days if any use it. This is a rename only; the next item proposes folding the event away. (src/analytics/event.ts:234)
   - **split status** `token_launcher.wallet_load_failed` → `token_launcher.creation_failed`: This is a _failed event for the token-creation flow, where the rules put success and failure on one outcome event as status and failure_reason. token_launcher.creation_failed is already a second _failed event. Propose one outcome event with status = success | failed | cancelled and failure_reason = hardware_wallet_unsupported | private_key_unavailable | not_authenticated | exception. The hardware-wallet branch is a client-side refusal with no request formed, so it could go on error_occurred with feature, action and error_reason instead. Biometric cancel currently emits nothing and could become status = cancelled. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:61)
   - **privacy** `token_launcher.share_pressed`: The call spreads getAnalyticsParams(), which sends the raw token name, symbol, description and logo URL typed by the person, plus the address and url fields. name and address match the never-send patterns. Replace the free text with lengths or booleans (for example has_description, name_char_count) and keep the identifiers only if a report needs them. (src/screens/token-launcher/state/tokenLauncherStore.ts:243-257)
   - **rename** `top_holders.shown` → `top_holders_shown`: A dot in the name breaks the naming pattern. Dual-fire for 30 days if it is on a dashboard. (src/analytics/event.ts:338)
   - **rename** `Viewed profile` → `viewed_profile`: The name is Title case with a space, which breaks the snake_case pattern. Dual-fire for 30 days if it is on a dashboard. (src/analytics/event.ts:52)
   - **privacy** `Viewed profile.name`: The property is named name, which matches the never-send patterns. It holds the ENS name of another person's profile, which identifies a person. Replace it with a boolean such as has_ens_name, or drop it. (src/screens/ProfileSheet.tsx:71)
   - **merge** `Viewed profile` → `Viewed ENS profile`: Two names look like the same action of viewing a profile: 'Viewed profile' and 'Viewed ENS profile'. Confirm what the second one tracks before merging. If they overlap, keep one event with a property that tells the sources apart. (src/analytics/event.ts:51-52)
   - **property** `Show Secret View`: Back up wallet manually (view secret warning) on `Routes.VIEW_WALLET_BACKUP` (core) has no event: onManualBackup sends the person to SECRET_WARNING with isBackingUp, and the same secret view later renders. Adding isBackingUp to Show Secret View separates manual backups from plain secret views. Backup Complete already records the outcome, so no new event is needed. Add `isBackingUp`, `backupType`. (src/screens/SettingsSheet/components/Backups/ViewWalletBackup.tsx#ViewWalletBackup)
   - **add event** `wallet_switched`: Switch wallet on `Routes.CHANGE_WALLET_SHEET` (core) has no event. Answers how often people switch between wallets and how many use more than one. Fires in onChangeAccount only when the switch actually proceeds, which excludes edit mode and a tap on the current address. The sheet's other events are taps on menu items, and none records the switch itself. Properties: `entry_point`, `wallet_count`, `is_watch_only_target`, `target_wallet_type`, `is_pinned`. (src/screens/change-wallet/ChangeWalletSheet.tsx#ChangeWalletSheet)
   - **add event** `walletconnect_session_request_resolved`: Cancel a WalletConnect connection on `Routes.WALLET_CONNECT_APPROVAL_SHEET` (secondary) has no event. Shown Walletconnect session request has no outcome, so approval rate and abandonment can't be measured. This pairs with it: handleConnect gives approved, handleCancel gives rejected, and the unmount cleanup with handled false gives dismissed. Cancelled and dismissed are not failures. Properties: `status`, `dappName`, `dappUrl`, `request_type`, `chainId`, `wait_time_seconds`, `wallet_changed`, `network_changed`. (src/screens/WalletConnectApprovalSheet.tsx#WalletConnectApprovalSheet)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
3. **hooks** (one PR)
   - **rename** `Error during Google Drive Backup` → `error_during_google_drive_backup`: Title-case name breaks the snake_case pattern. Better to merge it with the iCloud error into one backup outcome event (see the merge item); if the name is kept, dual-fire for a window if it is on a dashboard. (src/hooks/useWalletCloudBackup.ts:131)
   - **merge** `Error during iCloud Backup` → `wallet_backup_completed`: The Google Drive and iCloud errors are the same action on two platforms, and the label property already names the platform. Fold both and Error updating Backup status into one outcome event with status = success | failed | cancelled, failure_reason and a provider property (the label values). No success event exists today, so the failure rate can't be computed (rule 4). (src/hooks/useWalletCloudBackup.ts:131)
   - **merge** `Error updating Backup status` → `wallet_backup_completed`: A failure of the local save after a successful upload; fold into the outcome event as a failure_reason such as status_update_failed. Its name breaks the snake_case pattern. (src/hooks/useWalletCloudBackup.ts:152)
   - **rename** `iCloud not enabled` → `error_occurred`: This is a client-side refusal before any request exists (rule 4 exception). It belongs on error_occurred with feature=backup, action=start and error_reason=icloud_unavailable. Dual-fire while any dashboard uses it. The current name breaks the snake_case pattern. (src/hooks/useWalletCloudBackup.ts:56)
   - **remove** `Ignore how to enable iCloud`: Dismissing an informational alert: it breaks no question a product owner would ask (rules 1 and 'what to ignore'). Its sibling 'View how to Enable iCloud' has the same problem. The name also breaks the pattern. (src/hooks/useWalletCloudBackup.ts:74)
   - **property** `Imported seed phrase`: The drafted address, addressOrEnsName, ens, input and type are not sent by this event. They belong to watchWallet and the showWalletProfileModalFor* events in the same file. The event itself sends no properties, so it can't be broken down by import type (seed phrase, private key, address watch, hardware). (src/hooks/useImportingWallet.ts:337)
   - **rename** `Imported seed phrase` → `wallet_imported`: The name is title-case, and the success path also covers private keys, addresses and hardware, not just seed phrases. Add an import_type property and a status for failures. Failure of initializeWallet currently has no event, so the import intent (tappedImportButton) has no matching outcome. (src/hooks/useImportingWallet.ts:337)
   - **rename** `mined_transaction.assets_resolved` → `transaction_assets_resolved`: The dot in the name breaks the snake_case pattern. Merge with mined_transaction.assets_timed_out into one event with status = resolved | timed_out (rule 4). Both names are in the code now; dual-fire if either is on a dashboard. (src/hooks/useWatchAssetUpdates.ts:173)
   - **merge** `mined_transaction.assets_timed_out` → `transaction_assets_resolved`: A timeout is the failure outcome of the same watch. Fold into the resolved event as status=timed_out. The merged event should carry chainId, type and timeToResolve on both paths: today resolved sends only timeToResolve and timed-out sends only chainId and type. (src/hooks/useWatchAssetUpdates.ts:70)
   - **rename** `network_status.offline` → `network_status_changed`: The dot breaks the snake_case pattern. Offline and reconnected are one action with two states; merge into one event with status = offline | reconnected. Dual-fire if either name is on a dashboard. (src/hooks/useIsOffline.ts:47)
   - **merge** `network_status.reconnected` → `network_status_changed`: Same action as network_status.offline with the opposite state; fold in as status=reconnected. (src/hooks/useIsOffline.ts:39)
   - **rename** `pending_transaction.resolved` → `pending_transaction_resolved`: The dot breaks the snake_case pattern. It fires only for confirmed transactions; failed and dropped outcomes have no event. Add status and failure_reason so failures are counted (rule 4). (src/hooks/useWatchPendingTxs.ts:92)
   - **merge** `Scanned address QR code` → `qr_code_scanned`: There are four events for one action (Scanned address QR code, Scanned Rainbow profile url, Scanned WalletConnect QR code, Scanned broken or unsupported QR code). Use one qr_code_scanned with a scan_type property (address | profile | walletconnect | invalid), which would also remove the title-case names (rule 3). (src/hooks/useScanner.ts:57)
   - **property** `Scanned address QR code`: The draft lists qrCodeData on this event, but it is sent only by the invalid-QR event; this one sends no properties of its own. (src/hooks/useScanner.ts:57)
   - **privacy** `Scanned broken or unsupported QR code`: qrCodeData is the raw decoded content of an unrecognised QR code, which can be arbitrary text, URLs or personal data. Send a length (char_count) or a detected scheme instead of the content. (src/hooks/useScanner.ts:124)
   - **other** `Scanned broken or unsupported QR code`: A client-side refusal with no request. Under rule 4 it belongs on error_occurred (feature=qr_scanner, error_reason=unrecognized) unless it is merged into qr_code_scanned as scan_type=invalid. (src/hooks/useScanner.ts:124)
   - **property** `refresh_account_data`: duration includes the 1.25 s MIN_REFRESH_DURATION animation delay, so it measures what the user saw, not system latency. Document it as UX time or subtract the fixed delay. Add a name such as duration_ms to make the unit clear. (src/hooks/useRefreshAccountData.ts:59)
   - **rename** `Scanned Rainbow profile url` → `qr_profile_scanned`: breaks the snake_case object_action pattern; keep the old name until dashboards are checked, then dual-fire for 30 days if used (src/hooks/useScanner.ts:78)
   - **rename** `Scanned WalletConnect QR code` → `qr_walletconnect_scanned`: breaks the naming pattern; deprecate the old name, dual-fire window if on a dashboard. Better still, merge it with the other QR scan events into one qr_scanned event with a scan_type property (rule 3) (src/hooks/useScanner.ts:105)
   - **merge** `Scanned Rainbow profile url` → `qr_scanned`: The scan events for profile, WalletConnect, address and invalid QR codes are one action with different results. Fold them into one event with scan_type = profile | walletconnect | address | invalid (src/hooks/useScanner.ts:57,78,105,124)
   - **property** `Scanned Rainbow profile url.qrCodeData`: The draft lists qrCodeData on this event and on 'Scanned WalletConnect QR code', but the code never sends it on them. Only the broken-QR call at useScanner.ts:124 sends it, so remove it from both of these entries and keep it on the invalid-scan event (src/hooks/useScanner.ts:78,105,124)
   - **privacy** `Scanned broken or unsupported QR code.qrCodeData`: Raw scanned QR content is sent as a property. It can hold a seed phrase, a private key or a URL with tokens. Replace it with a length or a type bucket (src/hooks/useScanner.ts:124)
   - **merge** `Show wallet profile modal for ENS address` → `wallet_profile_modal_shown`: The four 'Show wallet profile modal for ...' events are one action with a different input kind. Merge them into one event with input_type = ens | unstoppable | address | seed_phrase (rule 3), and keep the shared address property (src/hooks/useImportingWallet.ts:170,200,222,279)
   - **rename** `Show wallet profile modal for ENS address` → `wallet_profile_modal_shown`: breaks the naming pattern; deprecate the old names with a dual-fire window if dashboards use them (src/hooks/useImportingWallet.ts:170)
   - **property** `Show wallet profile modal for ENS address.properties`: The draft lists addressOrEnsName, ens and type on this event and on the Unstoppable one, but the code sends only address and input. Remove the properties the code doesn't send. Likewise, the read-only event sends ens and input only, the imported event address and type only. addressOrEnsName belongs to watch_wallet (src/hooks/useImportingWallet.ts:163,170,200,222,279)
   - **other** `Show wallet profile modal for read only wallet`: The name says 'read only wallet' but the call fires for every plain valid 0x address in the flow, whether the user is importing or watching it. Use input_type = address in the merged event (src/hooks/useImportingWallet.ts:209-225)
   - **privacy** `Show wallet profile modal for ENS address.input`: input is the text the user typed (ENS name, Unstoppable domain or plain address), and 'address' is the full resolved wallet address, while the rest of the plan hashes wallet addresses (walletAddressHash). Decide whether raw addresses and names may be sent, or replace them with a hash or an input type (src/hooks/useImportingWallet.ts:170,200,222,279)
   - **rename** `Tapped "Import" button` → `wallet_import_tapped`: breaks the naming pattern (title case, quotation marks); deprecate the old name, dual-fire window if on a dashboard (src/hooks/useImportingWallet.ts:132)
   - **other** `Tapped "Import" button`: It fires before validation, so invalid taps count as intent, and the outcome event ('Imported seed phrase') exists only for successful imports and has no failure counterpart. Consider carrying status and failure_reason on the outcome (rule 4) so import drop-off can be measured (src/hooks/useImportingWallet.ts:132-136,337)
   - **rename** `Toggled an NFT as Hidden` → `nft_hidden_toggled`: breaks the naming pattern; deprecate the old name, dual-fire window if on a dashboard (src/hooks/useHiddenTokens.ts:34)
   - **property** `Toggled an NFT as Hidden.isHidden`: The draft types isHidden as string, but the code sends a boolean (true / false). Correct the type to boolean (src/hooks/useHiddenTokens.ts:34,73)
   - **rename** `View how to Enable iCloud` → `icloud_help_viewed`: breaks the naming pattern; deprecate the old name, dual-fire window if on a dashboard (src/hooks/useWalletCloudBackup.ts:66)
   - **property** `View how to Enable iCloud.properties`: The draft lists error and label here, but they come from the backup-error call at useWalletCloudBackup.ts:131 and are not sent by this event. Remove them from this entry (src/hooks/useWalletCloudBackup.ts:66,131)
   - **merge** `View how to Enable iCloud` → `cloud_backup_failed_help`: This event, 'iCloud not enabled' and 'ignore how to enable iCloud' are the choices at one alert. Consider one event with a choice property (show_me | no_thanks). The backup error events (errorDuringICloudBackup / errorDuringGoogleDriveBackup) should become status + failure_reason on a backup outcome event (rule 4) (src/hooks/useWalletCloudBackup.ts:56,66,74,131)
   - **rename** `Viewed ENS profile` → `ens_profile_viewed`: breaks the naming pattern; deprecate the old name, dual-fire window if on a dashboard (src/hooks/useOnAvatarPress.ts:111)
   - **property** `Viewed ENS profile.from`: from is a hard-coded 'Transaction list' that names the entry point; use the shared vocabulary name entry_point. The hook is shared, so the value may be wrong from other screens. Also category is a constant ('profiles') on this event, which adds nothing (src/hooks/useOnAvatarPress.ts:111-115)
   - **privacy** `Viewed ENS profile.ens`: The user's own ENS name is sent as a property, alongside walletAddressHash. Check whether it's needed; a has_ens boolean would answer the question (src/hooks/useOnAvatarPress.ts:111-115)
   - **property** `watch_wallet.ens`: The draft lists `ens`, `input` and `type`, but `watch_wallet` never sends them. They belong to `showWalletProfileModalForReadOnlyWallet` (`ens`, `input`), `showWalletProfileModalForENSAddress` (`address`, `input`) and `showWalletProfileModalForImportedWallet` (`address`, `type`). The scanner attributed those neighbouring calls to this event, so drop `ens`, `input` and `type` from the entry. The `watch_wallet` calls are only at lines 163, 193, 233 and 271, and the 132 and 170 hits are other events. The code sends only `addressOrEnsName` and `address`, plus the automatic wallet context. (src/hooks/useImportingWallet.ts:163, 193, 233, 271; src/analytics/event.ts:814-817)
   - **privacy** `watch_wallet.address`: The raw wallet address (`address`) and the ENS or domain name (`addressOrEnsName`) go out as event properties. `address` matches the never-send pattern `^address$`, and the app hashes addresses elsewhere (`walletAddressHash`) to avoid sending them. Send a hash, or just the input kind (ens | unstoppable | address), instead of the raw values. (src/hooks/useImportingWallet.ts:163-166, 233-236)
   - **add pair** `watch_wallet`: The event fires before the wallet is actually added (before `startImportProfile`), and the later steps can fail or be abandoned. There is no outcome event for a watched wallet being saved. Consider a `status` (success | failed | cancelled) on an outcome event, or confirm the existing import-success events cover watch wallets. (src/hooks/useImportingWallet.ts:163-169, 299-338)
   - **merge** `watch_wallet`: The three resolution branches each fire their own `showWalletProfileModalFor…` event (ENS, Unstoppable, ReadOnlyWallet, ImportedWallet) alongside `watch_wallet`. These are one action with a different input kind. They should be one event with an `input_kind` property. Each of them also sends the raw address and input. (src/hooks/useImportingWallet.ts:170, 200, 222, 279)
   - **property** `watch_wallet.addressOrEnsName`: Property names are camelCase (`addressOrEnsName`, `walletAddressHash`) while the event name is snake_case. Renaming the property would break existing data, so keep it, but decide if the vocabulary should move to snake_case. (src/analytics/event.ts:814-817)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
4. **swaps** (one PR)
   - **rename** `swaps.changed_chain_id` → `swap_chain_changed`: name contains a dot, breaking the snake_case pattern; keep old name until dashboards move, then dual-fire window if used (src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:75)
   - **rename** `swaps.flipped_assets` → `swap_assets_flipped`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/components/FlipButton.tsx:97)
   - **rename** `swaps.received_quote` → `swap_quote_received`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438)
   - **rename** `swaps.searched_for_token` → `swap_token_searched`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:173)
   - **rename** `swaps.selected_asset` → `swap_asset_selected`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023)
   - **rename** `swaps.toggled_degen_mode` → `swap_degen_mode_toggled`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/hooks/useSwapSettings.ts:14)
   - **rename** `swaps.submitted` → `swap_submitted`: name contains a dot, breaking the naming pattern (src/__swaps__/screens/Swap/providers/swap-provider.tsx:396)
   - **split status** `swaps.failed` → `swaps.submitted`: a separate _failed event for the swap outcome; failure should be status = success | failed plus failure_reason on the outcome event, so conversion rates don't need a union. Same properties are already sent by both via trackSwapEvent. swaps.succeeded (in event.ts, outside this shard) is a third outcome event for the same swap (src/__swaps__/screens/Swap/providers/swap-provider.tsx:342)
   - **split status** `swaps.quote_failed` → `swaps.received_quote`: a _failed event for the quote request; fold into the quote event as status + failure_reason (error_code/reason). swaps.received_quote already fires for error quotes (src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:47)
   - **property** `swaps.quote_failed`: the code sends error_code, reason, inputAsset, outputAsset, inputAmount, outputAmount but the draft lists only walletAddressHash and walletType; add those six properties (inputAsset/outputAsset as { address, chainId, symbol } objects, inputAmount/outputAmount numeric, reason is the error message string) (src/__swaps__/screens/Swap/hooks/analyticsTrackQuoteFailed.ts:38-47)
   - **property** `swaps.selected_asset`: draft lists 20 properties (degenMode, errorMessage, inputAsset*, outputAsset*, isBridge, ..., parameters) that this call does not send; they come from trackSwapEvent. The call sends only asset, otherAsset, type. Remove them from the entry (src/__swaps__/screens/Swap/providers/swap-provider.tsx:1023)
   - **property** `swaps.submitted`: draft lists asset, otherAsset (sent only by swaps.selected_asset) and parameters (never sent; trackSwapEvent only reads it) — remove them. The same applies to swaps.failed. The scanner attributed all three events to swap-provider.tsx:342, :396 and :1023 although each call belongs to one event (src/__swaps__/utils/trackSwapEvent.ts:47-71)
   - **property** `swaps.failed`: draft lists asset, otherAsset and parameters, none of which are sent by trackSwapEvent; remove them (src/__swaps__/utils/trackSwapEvent.ts:47-71)
   - **other** `token_list`: scanner attributed SendSheet.tsx:357 and :378 to this event, but those calls send changedNativeCurrencyInputSend and changedTokenInputSend. The real token_list sends in SendSheet.tsx:670 and useWalletSectionsData.ts:125 (screen 'send' and 'wallet') were not seen by the scanner; also add them to code_refs (src/features/transfer/screens/SendSheet.tsx:357,378,670; src/hooks/useWalletSectionsData.ts:125)
   - **property** `token_list`: the discover call computes no_price as the count of assets WITH a numeric price (`!isNaN(price.value)`), while the send and wallet calls count assets WITHOUT one; fix the discover condition so the property means the same everywhere (src/components/Discover/DiscoverSearch.tsx:115)
   - **privacy** `token_list.query`: raw user search text sent as an event property (matches never-send pattern ^query$); replace with char_count (src/__swaps__/screens/Swap/hooks/useSearchCurrencyLists.ts:155; src/components/Discover/DiscoverSearch.tsx:111)
   - **other** `swaps.selected_asset`: fires on every setAsset call, including when nothing changed and when asset is null, and sends whole asset objects; consider firing only when didSelectedAssetChange and sending flat fields (src/__swaps__/screens/Swap/providers/swap-provider.tsx:1004-1027)
   - **add pair** `swaps.submitted`: swaps.submitted fires from the sheet's confirm; there is no intent event for the swap review/confirm tap, so abandoned swaps at the review step can't be measured, only quotes received. Needs a decision before adding (event budget gate) (src/__swaps__/screens/Swap/providers/swap-provider.tsx:410-421)
   - **add event** `perps_deposit_completed`: Deposit funds into perps account on `Routes.PERPS_DEPOSIT_SCREEN` (core) has no event. The perps deposit flow (PerpsDepositScreen using the shared DepositScreen and useDepositHandler) has no analytics, while withdrawals already have perps.withdrew and perps.withdraw.failed. This answers how many deposits succeed or fail, with what funding asset or chain, and whether sponsored gas helps. A deposit can fail or be abandoned, so a matching intent event (perps_deposit_started) is worth adding too. Send both from the perps-specific path, not from the shared funding system, so they aren't mixed with other deposit configs. Properties: `status`, `failure_reason`, `failure_stage`, `amount_usd`, `from_asset_symbol`, `from_chain_id`, `execution_strategy`, `is_sponsored`, `is_first_deposit`. (src/features/perps/screens/perps-deposit-withdraw-screen/PerpsDepositScreen.tsx#PerpsDepositScreen)
   - **add event** `delegation_revoked`: Revoke smart wallet delegation on `Routes.REVOKE_DELEGATION_PANEL` (core) has no event. handleRevoke only logs, so nobody can count how many users disable their smart wallet or respond to a vulnerability alert, or how often it works. This is a money-safety flow where the number answers whether security alerts get acted on. The panel opening (intent) is worth adding too as delegation_revoke_started, so you can see users who open it and never confirm. Properties: `status`, `failure_reason`, `revoke_reason`, `is_backend_triggered`, `chain_ids_count`, `failed_chain_count`, `is_retry`, `attempt_number`. (src/features/delegation/screens/RevokeDelegationPanel.tsx#RevokeDelegationPanel)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
5. **asset_list** (one PR)
   - **rename** `claimable_panel.opened` → `claimable_panel_opened`: The dot breaks the object_action snake_case pattern. Keep the old name in the registry, deprecate it, and dual-fire only if it is on a dashboard. (src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88)
   - **property** `claimable_panel.opened`: The draft types assets as string and chainId as string, but the code sends an array of objects and a numeric ChainId. Fix the draft types. (src/components/asset-list/RecyclerAssetList2/Claimable.tsx:88-98)
   - **rename** `Pressed Button` → `button_pressed`: Title-case with spaces breaks the pattern. It is also the generic catch-all for button presses. (src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23)
   - **merge** `Tapped "Add Cash"` → `button_pressed`: Add Cash, Swap, Send and My QR Code are the same kind of action (a home-screen button tap) sent as four names with the same properties. Fold them into one event with a button name property (rule 3) and dual-fire if any is on a dashboard. Their names also break the naming pattern. (src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:140)
   - **merge** `Tapped "Swap"` → `button_pressed`: Same as Add Cash: four home-screen button events folded into one with a button name property. (src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:157)
   - **merge** `Tapped "Send"` → `button_pressed`: Same as Add Cash: four home-screen button events folded into one with a button name property. (src/components/asset-list/RecyclerAssetList2/profile-header/ProfileActionButtonsRow.tsx:174)
   - **merge** `Tapped "My QR Code"` → `button_pressed`: Same as Add Cash: four home-screen button events folded into one with a button name property. (src/components/asset-list/RecyclerAssetList2/index.tsx:102)
   - **other** `Pressed Button`: The scanner lists ChangeWalletSheet.tsx lines 264, 267 and 286 as Pressed Button calls, and Add Cash, Swap and Send each at three lines of ProfileActionButtonsRow.tsx. Those lines send other events (tappedDoneEditingWallet, tappedCancelEditingWallet, tappedEditWallet, and navigationAddCash / navigationSwap / navigationSend at their own lines). The only real Pressed Button calls here are DiscoverMoreButton.tsx:23 and ChangeWalletSheet.tsx:397. The scanner's property keys for those events are wrong, and the draft's wallet_label property should not be on Pressed Button. The scanner also shows each call twice. (src/screens/change-wallet/ChangeWalletSheet.tsx:264,267,286,397)
   - **property** `Pressed Button.wallet_label`: The draft lists wallet_label but Pressed Button does not send it. Remove it from this entry. It belongs to the wallet-edit event (tappedDoneEditingWallet), which is outside this shard. (src/screens/change-wallet/ChangeWalletSheet.tsx:264)
   - **property** `Pressed Button`: The draft marks buttonName and action as optional, but both real call sites always send them. Make them required. (src/components/asset-list/RecyclerAssetList2/core/DiscoverMoreButton.tsx:23-26)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
6. **expanded_state** (one PR)
   - **rename** `add_new_wallet_group.name` → `wallet_group_name_submitted`: Breaks the snake_case pattern because of the dot. The event is also not on any known dashboard, so a short dual-fire window is enough if analysts query it. (src/analytics/event.ts:184)
   - **privacy** `add_new_wallet_group.name`: The 'name' property sends the raw text the user typed as the wallet group name, which matches the never-send pattern ^name$. Replace it with char_count, computed from value.trim().length. (src/components/expanded-state/NewWalletGroupState.tsx:20)
   - **other** `add_new_wallet_group.name`: The event fires on the submit tap before the group exists, with no outcome event. Rule 2 wants an intent/outcome pair, so consider an outcome event or a success marker. Either way, an empty name currently fires it as well. (src/components/expanded-state/NewWalletGroupState.tsx:19)
   - **rename** `Tapped "Cancel" on Wallet Profile modal` → `wallet_profile_cancelled`: Breaks the snake_case pattern: the name is a sentence with quotes and capitals. Dual-fire for a window if it is on a dashboard. (src/analytics/event.ts:292)
   - **rename** `Tapped "Submit" on Wallet Profile modal` → `wallet_profile_submitted`: Breaks the snake_case pattern: the name is a sentence with quotes and capitals. Dual-fire for a window if it is on a dashboard. (src/analytics/event.ts:293)
   - **add pair** `Tapped "Submit" on Wallet Profile modal`: Submit and cancel are tracked, but there is no event for opening the modal and none for the save outcome (the keychain callback for Create). Abandonment and failure after the tap cannot be measured. Consider an open event carrying action_type (Create | edit) and entry point. Do not fire the submit event from two places. (src/components/expanded-state/WalletProfileState.js:60)
   - **property** `Tapped "Cancel" on Wallet Profile modal`: The call sends no properties of its own, so analysts cannot split cancel and submit by actionType (Create or edit) or isFromSettings. Both values are in scope in the handlers. Add them as properties. (src/components/expanded-state/WalletProfileState.js:54)
   - **rename** `token_details.nft` → `nft_details_viewed`: Breaks the snake_case pattern because of the dot. The new name also states what happened. Dual-fire for a window if it is on a dashboard. (src/analytics/event.ts:222)
   - **property** `token_details.nft`: The call sends a nested 'token' object (isPoap, isParty, isENS, address, chainId, name, image_url) that the draft entry lacks. Add 'token' to the entry, or flatten it into separate properties. The scanner's key list omitted it. (src/components/expanded-state/UniqueTokenExpandedState.tsx:391)
   - **privacy** `token_details.nft`: token.name is the NFT's name, which for ENS NFTs is the user's ENS name, and token.image_url is a URL. Both go out as event properties. Check whether they identify a person, and if so replace them with is_ens and the contract address and chain. (src/components/expanded-state/UniqueTokenExpandedState.tsx:391)
   - **other** `token_details.nft`: The event fires from a 5-second timer or from unmount, and the call ignores the hook's 'cancelled' flag. Pass the flag as a property, or fire on mount, so that views shorter than 5 seconds can be told apart. (src/hooks/useTimeout.ts:40)
   - **add event** `contact_saved`: Save or edit a contact profile on `Routes.MODAL_SCREEN` (secondary) has no event. This answers how many contacts people save per week and from where. ContactProfileState.handleAddContact saves a contact only when the nickname is non-empty, and nothing records it today. It isn't a Wallet Profile modal tap, so no existing event covers it. Cancel and abandonment aren't worth a separate event: the modal has no start event, and an empty name never reaches the save. Properties: `entry_point`, `is_new_contact`, `has_ens`, `nickname_char_count`, `walletAddressHash`. (src/screens/ModalScreen.js#ModalScreen)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
7. **cards** (one PR)
   - **rename** `buy_button.pressed` → `buy_button_pressed`: The dot breaks the object_action snake_case pattern. Keep the old name firing alongside the new one for a dual-fire window if it feeds any dashboard. (src/analytics/event.ts:91)
   - **rename** `card.pressed` → `card_pressed`: The dot breaks the object_action snake_case pattern. Keep the old name firing alongside the new one for a dual-fire window if it feeds any dashboard. (src/analytics/event.ts:87)
   - **rename** `qr_code.viewed` → `qr_code_viewed`: The dot breaks the snake_case pattern. The event also fires on the tap, not when the modal is shown, so the name misleads. A name such as qr_code_pressed would be more accurate, but that is a product call. (src/analytics/event.ts:90; src/components/cards/ReceiveAssetsCard.tsx:46-51)
   - **property** `buy_button.pressed`: The draft lists cardName and cardType, but no buy_button.pressed call sends them. The scanner appears to have attributed the card.pressed call at EthCard.tsx:90 to this event. Remove both from this entry. The type in event.ts also allows amount and newWallet, which no call sends. (src/components/cards/EthCard.tsx:74-77, :90-94; src/analytics/event.ts:468-472)
   - **property** `card.pressed`: The draft lists componentName, but no card.pressed call sends it. The scanner appears to have attributed the buy_button.pressed call at EthCard.tsx:74 to this event. Remove it from this entry. (src/components/cards/EthCard.tsx:74-77, :90-94)
   - **other** `card.pressed`: The scanner missed a fourth call site. ENSSearchCard fires card.pressed at ENSSearchCard.tsx:54 with cardName, routeName and cardType ('square'). The scan output should include it. (src/features/ens/components/ENSSearchCard.tsx:54-58)
   - **add pair** `buy_button.pressed`: Buy / Add Cash is an intent with no outcome event in this code. The outcome probably arrives through an external provider. If it does, abandonment is intent with no matching outcome, so a server-side outcome event with status (success | failed | cancelled) would make drop-off measurable. (src/components/cards/EthCard.tsx:67-78)
   - **add event** `pin_completed`: Create and confirm a PIN on `Routes.PIN_AUTHENTICATION_SCREEN`; Enter PIN to authenticate on `Routes.PIN_AUTHENTICATION_SCREEN` (core) has no event. The question is how many people who start setting a PIN finish it, and how often the confirm step is mismatched. Fire it once when the screen finishes: on onSuccess, on onCancel (the unmount cleanup when `finished` is false, where cancelled is not a failure), or when the lockout forces a goBack. Never send the PIN itself. This screen is opened by a caller, so the cancelled status stands in for a separate intent event. Properties: `action`, `status`, `failure_reason`, `attempts_failed`, `walletType`. (src/features/local-auth/screens/PinAuthenticationScreen.js#PinAuthenticationScreen, src/features/local-auth/screens/PinAuthenticationScreen.js#PinAuthenticationScreen)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
8. **discover** (one PR)
   - **privacy** `Search Query.query`: Raw user-typed search text is sent as an event property; matches the never-send pattern ^query$. Stop sending it and keep length (and a result_count if needed). (src/components/Discover/DiscoverSearchInput.tsx:134-138)
   - **rename** `Search Query` → `discover_search_cleared`: Name breaks the snake_case object_action pattern, and it is fired when the input is cleared, not when a search runs. Check dashboards before renaming; dual-fire if it is used. (src/components/Discover/DiscoverSearchInput.tsx:134)
   - **rename** `Tapped Search` → `discover_search_opened`: Name breaks the naming pattern (capitalised with spaces). Dual-fire if it is on a dashboard. (src/components/Discover/DiscoverScreenContext.tsx:64)
   - **rename** `Time spent on the Discover screen` → `discover_screen_time_spent`: Name breaks the naming pattern (sentence with spaces). Dual-fire if it is on a dashboard. (src/components/Discover/useTrackDiscoverScreenTime.ts:22)
   - **property** `Search Query.length`: Draft types it as string but the code sends a number (searchQuery.length); type is numeric. (src/components/Discover/DiscoverSearchInput.tsx:136)
   - **property** `Time spent on the Discover screen.durationInMs`: Draft types it as string but the code sends a number (performance.now difference); type is numeric. (src/components/Discover/useTrackDiscoverScreenTime.ts:22-24)
   - **add pair** `Search Query`: Search is only recorded when the input is cleared with more than 1 character. Searches ended by cancel or by leaving the screen are not recorded, so the intent (Tapped Search) has no reliable matching outcome. Consider a settled-search event with char_count and result_count. (src/components/Discover/DiscoverSearchInput.tsx:132-142)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
9. **state** (one PR)
   - **rename** `performance.time_to_sign` → `performance_time_to_sign`: dots break the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ pattern; deprecate the old name. If it is on a dashboard, dual-fire for 30 days. (src/analytics/event.ts:205)
   - **rename** `performance.time_to_sign.operation` → `performance_time_to_sign_operation`: dots break the naming pattern; deprecate the old name, with a dual-fire window if on a dashboard (src/analytics/event.ts:206)
   - **split status** `wallet_initialization.failed` → `wallet_initialized`: failure is a separate _failed event; it should be status = success | failed plus failure_reason on one outcome event. The success path only records a performance measure (event.performanceInitializeWallet), so a wallet_initialized outcome event does not exist yet and would need adding (no start/end pair today). (src/state/wallets/initializeWallet.ts:190)
   - **rename** `wallet_initialization.failed` → `wallet_initialized`: if the status merge is not adopted, the dot in the name still breaks the naming pattern (src/analytics/event.ts:243)
   - **property** `performance.time_to_sign`: The draft lists operation, startTime, endTime, metadata, timeToCompletion and resultWasNullish on this event, but the call only sends walletAddressHash, walletType, screen, completedAt, elapsedTime, totalElapsedTime and error. The scanner merged the keys of both calls into both events. Those six properties belong only to performance.time_to_sign.operation. (src/state/performance/performance.ts:69)
   - **property** `performance.time_to_sign.operation`: The draft lists elapsedTime, totalElapsedTime and error on this event, but the call only sends the PerformanceLog keys (completedAt, screen, operation, startTime, endTime, metadata, timeToCompletion, resultWasNullish) plus the wallet context. Those three belong only to performance.time_to_sign. The error that made a step throw is therefore lost on the operation event. (src/state/performance/performance.ts:62)
   - **other** `performance.time_to_sign`: Nothing is sent for hardware wallets because isEnabled() returns false for them, so the event's metrics describe only software wallets. Document this in any dashboard built on it. (src/state/performance/performance.ts:30)
   - **add event** `predictions_buy_started`: Start buying a market outcome on `Routes.POLYMARKET_MARKET_SHEET` (core) has no event. A buy can be abandoned: only `predictions.place_order` and its failed event exist, with nothing before them. This answers 'how many people open a buy and how many complete it, by entry point?'. The same name is sent from every place that opens the new-position sheet, with `entry_point` to tell them apart. Fire it when the new-position sheet opens. Properties: `event_slug`, `market_slug`, `outcome`, `outcome_index`, `entry_point`. (src/features/polymarket/screens/polymarket-market-sheet/PolymarketMarketSheet.tsx#PolymarketMarketSheet)
   - **add event** `backup_restore_started`: Select a cloud backup to restore on `Routes.VIEW_CLOUD_BACKUPS` (core) has no event. Answers how many people start restoring from a cloud backup, which backup they pick and how old it is. Selecting a backup opens the restore step and the restore can fail or be abandoned, so pair it with a `backup_restore_completed` outcome (`status`, `failure_reason`) if the plan has none. Properties: `entry_point`, `is_most_recent`, `backup_age_bucket`, `cloud_platform`, `backup_count`. (src/screens/SettingsSheet/components/Backups/ViewCloudBackups.tsx#ViewCloudBackups)
   - **add event** `delegation_revoke_started`: Revoke a network delegation on `Routes.VIEW_WALLET_DELEGATIONS` (core) has no event. Answers how often users disable smart-account delegation, on one network or all, and whether they leave a third-party delegation or Rainbow's own. The code only navigates to the revoke panel and nothing is tracked in the delegation feature, so also add the outcome `delegation_revoked` (`status`, `failure_reason`) from the panel. `revoke_reason` covers the single-network, third-party and disable-smart-wallet paths. Properties: `revoke_reason`, `delegation_type`, `chain_id`, `entry_point`. (src/features/delegation/screens/ViewWalletDelegations.tsx#ViewWalletDelegations)
   - **add event** `wallet_error_recovery_selected`: Restore wallet from backup after wallet error on `Routes.WALLET_ERROR_SHEET` (core) has no event. Answers how many users who hit a broken keychain (passcode disabled, device changed) choose to restore versus dismiss. `action` is restore or dismiss, and `cause` and `resolution` come from the sheet's logic. The restore itself continues in the add-wallet flow. Properties: `cause`, `resolution`, `action`. (src/components/wallet-error/WalletErrorSheet.tsx#WalletErrorSheet)
   - **add event** `smart_wallet_toggled`: Enable or disable smart wallet on `Routes.VIEW_WALLET_DELEGATIONS` (secondary) has no event. It answers how many users turn the smart wallet on or off, and how many had active delegations to revoke when they did. Fire it where the preference actually changes: on delegation.enable/disable, and after the revoke success callback when disabling. Revoking has its own outcome in the revoke panel, so this event is not an intent/outcome pair. Properties: `action`, `address_hash`, `rainbow_delegation_count`, `revoked_networks`. (src/features/delegation/screens/ViewWalletDelegations.tsx#ViewWalletDelegations)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
10. **error_boundary** (one PR)
   - **rename** `error_boundary.viewed` → `error_boundary_viewed`: The dot breaks the naming pattern ^[a-z][a-z0-9]*(_[a-z0-9]+)+$. Deprecate the old name. Dual-fire only if it is on a dashboard. (src/analytics/event.ts:217)
   - **rename** `error_boundary.reset` → `error_boundary_reset`: The dot breaks the naming pattern. Deprecate the old name. Dual-fire only if it is on a dashboard. (src/analytics/event.ts:218)
   - **other** `error_boundary.viewed`: The event name says 'viewed' but it fires when an error is caught, not when a screen is opened. Intent/outcome here is crash screen shown then Restart tapped. Consider a name that says an error was shown (e.g. error_boundary_shown) when renaming. (src/app/error-boundary/ErrorBoundary.tsx:20)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
11. **model** (one PR)
   - **rename** `wallet_create.succeeded` → `wallet_created`: The dot in the name breaks the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ pattern. Add a status property and fold wallet_create.failed into this event, so the name carries no outcome suffix. Dual-fire the old and new names for a window if either is on a dashboard. (src/model/wallet.ts:722)
   - **split status** `wallet_create.failed` → `wallet_created`: Rule 4: failure belongs on the outcome event as status = success | failed with failure_reason, not on a separate failed event. Otherwise a creation success rate needs a union of two events. Rename error to failure_reason when merging. Keep the three fixed codes. Normalise the raw exception message into a bounded set of reasons. (src/model/wallet.ts:450, src/model/wallet.ts:498, src/model/wallet.ts:506, src/model/wallet.ts:730)
   - **property** `wallet_create.succeeded`: The call sends properties the draft lacks: isImported, isRestoring, isReadOnly, isHardwareWallet, isHDWallet, ethereumWalletType, walletLibraryType, silent, overwrite and hasCheckedWallet. They are typed in WalletCreateEventParams and send on every path. Add them to the registry entry, because they are what split a first-time create from an import or a restore. The draft also lists error on this event, but the code never sends it. (src/model/wallet.ts:439-446, src/model/wallet.ts:722, src/analytics/event.ts:396-407)
   - **property** `wallet_create.failed`: The failure calls send the same ten properties as wallet_create.succeeded (isImported, isRestoring, isReadOnly, isHardwareWallet, isHDWallet, ethereumWalletType, walletLibraryType, silent, overwrite, hasCheckedWallet), and the draft lacks them. Paths 2 to 4 send them. Path 1 also sends them, because analyticsParams is set before the wallet check. Add them to the registry entry. (src/model/wallet.ts:450-453, src/model/wallet.ts:498-501, src/model/wallet.ts:506-509, src/model/wallet.ts:730-733)
   - **add pair** `wallet_create.failed`: Exceptions thrown while deriving the account (the deriveAccountFromWalletInput call) happen before analyticsParams is set. No failure event is sent for them and they are logged as an error only. Failures there are invisible in the data. (src/model/wallet.ts:428-446, src/model/wallet.ts:729)
   - **add event** `wallet_account_created`: Add an account to an existing wallet group on `Routes.CHOOSE_WALLET_GROUP` (core) has no event. createAccountInExistingWallet never goes through createWallet, so wallet_create.succeeded and wallet_create.failed don't record it. This answers how many accounts users add to existing groups and how often it fails. The user can also cancel the naming modal, so an intent event is worth adding too. Properties: `status`, `failure_reason`, `wallet_type`, `account_count_after`, `entry_point`. (src/screens/ChooseWalletGroup.tsx#ChooseWalletGroup)
   - **add event** `transaction_replaced`: Speed up a pending transaction on `Routes.SPEED_UP_AND_CANCEL_BOTTOM_SHEET`; Attempt to cancel a pending transaction on `Routes.SPEED_UP_AND_CANCEL_BOTTOM_SHEET`; Speed up a pending transaction on `Routes.SPEED_UP_AND_CANCEL_SHEET`; Attempt to cancel a pending transaction on `Routes.SPEED_UP_AND_CANCEL_SHEET` (core) has no event. Speed-up sends a new transaction with a failure path (catch around sendTransaction), and nothing records it. The number shows how often pending transactions get stuck and whether the replacement works. A transaction_replace_started intent event is worth adding too, since users can close the sheet. Properties: `replacement_type`, `status`, `failure_reason`, `chain_id`, `tx_gas_fee_type`, `is_hardware_wallet`, `entry_point`. (src/screens/SpeedUpAndCancelSheet.tsx#SpeedUpAndCancelSheet, src/screens/SpeedUpAndCancelSheet.tsx#SpeedUpAndCancelSheet, src/screens/SpeedUpAndCancelSheet.tsx#SpeedUpAndCancelSheet, src/screens/SpeedUpAndCancelSheet.tsx#SpeedUpAndCancelSheet)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
12. **notifications** (one PR)
   - **rename** `Tapped Push Notification` → `push_notification_tapped`: Breaks the snake_case `object_action` naming pattern. Keep the old name in the code until it is confirmed whether it sits on a dashboard. If it does, dual-fire for 30 days, then deprecate the old name. (src/analytics/event.ts:101)
   - **rename** `Changed Global Notification Settings` → `global_notification_setting_changed`: Breaks the snake_case `object_action` naming pattern. Keep the old name in the code until it is confirmed whether it sits on a dashboard. If it does, dual-fire for 30 days, then deprecate the old name. (src/analytics/event.ts:100)
   - **property** `Tapped Push Notification.topic, Tapped Push Notification.action`: The draft lists `topic` and `action` on this event, but its only call (analytics.ts:11) sends just `campaign`. The scanner attributed both calls in analytics.ts to both events, so these two properties should be removed from the entry. Both `Tapped Push Notification` and `Changed Global Notification Settings` map to one fixed event name each (event.ts:100-101). (src/notifications/analytics.ts:11)
   - **property** `Changed Global Notification Settings.campaign`: The draft lists `campaign` on this event, but its only call (analytics.ts:17) sends `topic` and `action`. `campaign` belongs to the push-tap event only, so it should be removed from the entry. (src/notifications/analytics.ts:17)
   - **property** `Tapped Push Notification.campaign`: The value is the push payload's `data.type` (transaction | wc | marketing | default), not a campaign. Rename the property or add `notification_type` alongside it. Check `_properties.yaml` for an existing vocabulary name first. (src/notifications/analytics.ts:12)
   - **other** `Changed Global Notification Settings`: The event fires only on success, so failed publishes (which show an error alert) are invisible. If the toggle stays, add `status` (success | failed) and `failure_reason` to the same event. Today `GlobalNotificationTopic` is `{}`, so the event has no topic to report and may be dead code. Decide whether to keep or remove it. (src/notifications/settings/constants.ts:11)
   - **property** `Changed Global Notification Settings`: Turn notifications on or off for a wallet on `Routes.WALLET_NOTIFICATIONS_SETTINGS` (secondary) has no event: The master switch in `toggleAllowNotifications` is the same kind of action as the existing event, a settings change with subscribe or unsubscribe. A new `scope` property and a `topic` of `all` answer how many wallets turn notifications off, split by owned and watched. Fire it only after `toggleGroupNotifications` succeeds. The name says 'Global', so rename it if the plan allows. Add `scope`, `topic`, `action`, `walletAddressHash`, `walletType`. (src/screens/SettingsSheet/components/WalletNotificationsSettings.tsx#WalletNotificationsSettings)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
13. **performance** (one PR)
   - **rename** `performance.report` → `performance_report`: the dot breaks the snake_case naming pattern. Keep the old name in the code until it is confirmed whether any dashboard uses it, and dual-fire for a window if one does. (src/analytics/event.ts:246)
   - **rename** `Performance Wallet Initialize Time` → `performance_wallet_initialized`: Title Case with spaces breaks the snake_case naming pattern. Deprecate the old name, and dual-fire for a window if it is on a dashboard. (src/analytics/event.ts:247)
   - **property** `Performance Wallet Initialize Time.walletStatus`: The code sends walletStatus (new | imported | old) through finishMeasuring params, and the event schema declares it. The draft lacks it. Add it as a string property. (src/state/wallets/initializeWallet.ts:173; src/analytics/event.ts:948)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
14. **site** (one PR)
   - **remove** `debug.identify_probe`: Temporary diagnostic that its own comments say to remove after FEPLAT-67. It answers no product question (rule 1). Deprecate it in the registry and remove both calls when the ticket closes. (src/App.tsx:177, src/redux/settings.ts:121)
   - **rename** `debug.identify_probe` → `debug_identify_probe`: Contains a dot, so it breaks the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ pattern. Renaming only makes sense if the probe stays; removal is the better outcome. (src/analytics/event.ts:85)
   - **rename** `First App Open` → `first_app_open`: Title-case with spaces breaks the snake_case pattern. Deprecate the old name. If it is on a dashboard, dual-fire for 30 days. (src/App.tsx:198)
   - **property** `First App Open`: The draft lists probe and value, but the call at src/App.tsx:198 sends neither. The scanner attributed them from the neighbouring probe call. Remove both from this entry. The wrapper adds walletAddressHash and walletType, and also device_brand, device_manufacturer and device_model on Android and appsflyer_id when available. The draft lacks the last four. (src/App.tsx:177,198; src/analytics/index.ts:102-108)
   - **property** `debug.identify_probe`: The wrapper also adds device_brand, device_manufacturer, device_model on Android and appsflyer_id when available to every event. The draft doesn't list them. Consider listing them once as super properties. (src/analytics/index.ts:102-108,145-158)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
15. **value_chart** (one PR)
   - **rename** `Changed Chart Type` → `chart_type_changed`: breaks the snake_case object_action pattern; keep the old name and dual-fire for 30 days if it is on a dashboard (src/analytics/event.ts:314)
   - **rename** `Changed Candle Resolution` → `candle_resolution_changed`: breaks the snake_case object_action pattern; keep the old name and dual-fire for 30 days if it is on a dashboard (src/analytics/event.ts:315)
   - **property** `Changed Chart Type`: The draft lists candleResolution on this event, but the code only sends chartType (plus walletAddressHash and walletType from the default metadata). The draft should not carry candleResolution here. (src/components/value-chart/Chart.tsx:272)
   - **property** `Changed Candle Resolution`: The draft lists chartType on this event, but the code only sends candleResolution (plus walletAddressHash and walletType from the default metadata). The draft should not carry chartType here. (src/components/value-chart/Chart.tsx:283)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
16. **handlers** (one PR)
   - **rename** `f2c.provider.flow_completed` → `f2c_provider_flow_completed`: Contains dots, so it breaks the ^[a-z][a-z0-9]*(_[a-z0-9]+)+$ naming pattern. The same applies to its siblings f2c.provider.flow_opened, f2c.provider.flow_errored and f2c.transaction_received. Rename them together and deprecate the old names. Dual-fire if any is on a dashboard. (src/analytics/event.ts:112)
   - **other** `f2c.provider.flow_completed`: The code comment says SDK-callback providers such as Ratio fire this event from their callbacks, but only the two deeplink calls were found. Either the call is missing from the scan or the comment is stale. (src/handlers/deeplinks.ts:160)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
17. **resources** (one PR)
   - **property** `add_favorite_token.address`: The name `address` matches the never-send PII pattern ^address$. Here it is a public token contract address, not a person's address. Rename it to `token_address` so it can't be confused with a wallet address and passes the PII check. (src/resources/favorites.ts:179)
   - **property** `add_favorite_token.name`: The name `name` matches the never-send PII pattern ^name$. Here it is the token's name from metadata. Rename it to `token_name` so it passes the check. (src/resources/favorites.ts:181)
   - **property** `add_favorite_token.symbol`: For consistency with `token_address` and `token_name`, rename `symbol` to `token_symbol`. Do this in the same change as the other two renames. (src/resources/favorites.ts:182)
   - **add pair** `add_favorite_token`: Un-favoriting a token sends no event, so the favorite count only ever goes up and net favorites can't be measured. Add this only if someone asks that question. A `remove_favorite_token` event, or a `favorite_token_toggled` event with an `action = added | removed` property, would cover it. Otherwise leave it as it is. (src/resources/favorites.ts:174-176)
18. **utils** (one PR)
   - **rename** `app_store_review.prompted` → `app_store_review_prompted`: The dot breaks the object_action snake_case pattern ^[a-z][a-z0-9]*(_[a-z0-9]+)+$. Keep the old name in the registry as deprecated. Dual-fire for a window if it is used on a dashboard. (src/analytics/event.ts:303)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
19. **other** (one PR)
   - **property** `claim_claimable.succeeded.errorMessage`: The draft lists errorMessage and failureStep on succeeded, but no succeeded call sends them. Remove them from this entry. Also, claimableId is claimable.type in some flows and claimable.uniqueId in the airdrop flow, so it means two different things. (src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:469)
   - **privacy** `claim_claimable.failed.errorMessage`: The airdrop flow sends the raw exception message (e.message) as errorMessage. It is free-form text from the provider or wallet. Send a failure_reason code from a fixed list instead. (src/screens/Airdrops/utils.ts:139)
   - **property** `claim_claimable.failed.usdValue`: usdValue is the claimable's totalCurrencyValue.amount, which is in the person's display currency, not necessarily USD. The name is misleading. (src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:82)
   - **property** `add_wallet_flow.started.type`: Restore from cloud sends 'seed', although the schema has a 'backup' value for it, so cloud and seed restores can't be told apart. Consider sending 'backup'. (src/screens/AddWalletSheet.tsx:82)
   - **privacy** `rewards.pressed_leaderboard_item.ens`: ens is another person's ENS name, sent as an event property on a tap of someone else's leaderboard row. It identifies a person; replace with a boolean has_ens or drop it. (src/screens/rewards/components/RewardsLeaderboardItem.tsx:81)
   - **property** `Tapped "Cancel" after editing wallet / Tapped "Delete Wallet" (final confirm) / Tapped "Done" after editing wallet / Tapped "Edit Wallet"`: The drafts list action and buttonName (from pressed_button at line 397) and wallet_label on every wallet event, and the scanner attributed the lines 264/267/286/295 calls to all four. In the code, wallet_label is sent only by the Done event, and action/buttonName by none of these four. Remove action, buttonName from all four and wallet_label from Cancel, Delete confirm and Edit. (src/screens/change-wallet/ChangeWalletSheet.tsx:397)
   - **property** `walletAddressHash / walletType (all events in this shard)`: Added by the analytics wrapper from the active address on every event; omitted when no address exists (e.g. welcome screen). The drafts mark them required, but the code makes them optional. Mark required: false, or move them to super properties. (src/analytics/getWalletContext.ts:22)
   - **property** `token_launcher.* address`: The property is the launched token's public contract address, but the name 'address' matches the never-send pattern ^address$. Rename it to token_address so reviewers don't have to guess whether it is a user address. Applies to token_created and share_pressed. (src/screens/token-launcher/state/tokenLauncherStore.ts:380)
   - **privacy** `token_launcher getAnalyticsParams`: getAnalyticsParams returns name, symbol, description, logoUrl and links, all typed in by the user. It is spread into abandoned, token_created, creation_failed and share_pressed. name matches the never-send pattern ^name$. Replace them with lengths or booleans (name_length, has_description, has_logo, link_count), and keep chainId, totalSupply, extraBuyAmount and imageModerated. (src/screens/token-launcher/state/tokenLauncherStore.ts:243-257)
   - **privacy** `token_launcher.creation_failed.error`: error is the raw error.message from the SDK, chain client or RPC. It may include addresses, calldata or URLs. Send a bounded failure_reason enum instead and keep the message in the logger only. (src/screens/token-launcher/state/tokenLauncherStore.ts:440)
   - **property** `token_launcher.wallet_load_failed.address`: The draft lists address and url on this event, but no wallet_load_failed call sends them. The scanner attributed them from the token_launcher.share_pressed call, which spreads getAnalyticsParams(). Remove both from this entry. Also check that token_launcher.share_pressed has its own entry, since it wasn't in this shard. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:212)
   - **privacy** `token_launcher.wallet_load_failed.error`: One branch sends error.message from an arbitrary caught exception. It is free text and could include addresses or RPC details. Map it to a bounded failure_reason enum and log the message in the logger only. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:120)
   - **property** `top_holders.shown.chainId`: The draft types chainId as string, but the call and the event schema send a number. Use numeric. (src/screens/expandedAssetSheet/components/sections/TopHoldersSection.tsx:44)
   - **property** `Scanner attribution`: The scanner pooled keys from different events behind the shared analytics.track call, so draft properties don't match what each event sends. Examples: cash.buy_* got all four call sites' keys, 'Approved new WalletConnect session' got connector/reason/type, browser.tap_favorite got hasClickedBefore/index, and cash.amount_entered got reason. Properties each event doesn't send are marked required:false and described as not sent. Code should remove them from the drafts, and add image to browser.tap_favorite, which passes the whole FavoritedSite object. (src/features/wallet-connect/handlers/onSessionProposal.ts:158; src/features/dapp-browser/components/Homepage.tsx:226; src/features/cash/stores/cashBuyOrderStore.ts:123)
   - **property** `backup.error.error`: the value is a translated, user-facing message that varies by app language, so it won't group. Send a stable error code taken from CLOUD_BACKUP_ERRORS instead. (src/features/backup/backup.ts:184)
   - **privacy** `browser.*.name`: `name` matches the never-send pattern ^name$. Here it is a site/dapp name, not a person's name, but the name is derived from user-entered URLs. Rename to dapp_name, or confirm it is exempt. (src/features/dapp-browser/components/Homepage.tsx:323)
   - **property** `cash.card_linked.reason`: draft lists reason but cash.card_linked never sends it; the scanner took it from the cash.card_link_failed call at line 58. Remove it from the draft, and mark brand required. (src/features/cash/stores/cardLinkFlowStore.ts:47,58)
   - **property** `cash.existing_account_recovery_selected.mode, outcome, reason`: the draft lists mode, outcome and reason, but the call sends no properties. The scanner took them from cash.phone_submitted, cash.phone_already_registered and cash.phone_submit_failed calls in the same file. Remove them. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:190 (call) vs 68, 78, 126, 134)
   - **property** `cash.kyc_approved.mode, reason, source`: the draft lists mode, reason and source, but cash.kyc_approved sends no properties. They belong to neighbouring calls (phone_verified, phone_verify_failed, kyc_awaiting_decision). Remove them. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:227; kycStatusService.ts:22)
   - **property** `cash.kyc_awaiting_decision.mode, reason`: the draft lists mode and reason, but only source is sent. Remove mode and reason. The scanner also missed the kycStatusService.ts:23 call site (source = resume | return). (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:155; src/features/cash/services/kycStatusService.ts:23)
   - **property** `cash.kyc_failed.mode, source`: the draft lists mode and source, but only reason is sent. Remove them. The scanner also missed the calls at useSubmitReviewFlow.ts:191,231,235 and kycStatusService.ts:24-25. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:191,231,235)
   - **property** `cash.kyc_submitted.mode, reason, source`: the draft lists mode, reason and source, but the call at line 176 sends no properties. Remove them. The scanner missed that call and attributed the neighbouring ones. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:176)
   - **property** `cash.passkey_added.reason`: the draft lists reason but cash.passkey_added sends nothing beyond the wrapper defaults; reason belongs to cash.passkey_failed. Remove it. (src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:55,73)
   - **other** `Scanner attribution in useSubmitReviewFlow.ts / useAddPasskeyFlow.ts / useSubmitPhoneFlow.ts / cardLinkFlowStore.ts`: The scanner's call sites for several of these events are other events' calls. For example, the kyc_* events were given lines 91, 101, 124 (cash.phone_verified, cash.phone_verify_failed, cash.phone_submitted) and passkey_added was given line 47 (cash.passkey_submitted) and line 73 (passkey_failed). It probably read the call sites against the wrong event. Re-run the scan after fixing so each event's code_refs are correct. (useSubmitReviewFlow.ts:91,101,124,155,176,191,227; useAddPasskeyFlow.ts:47,55,73)
   - **property** `cash.passkey_submitted.reason`: the draft lists reason on this event but no call passes it; the scanner picked it up from the cash.passkey_failed call at :73. Remove reason from the draft of this event. (src/features/cash/screens/cash-deposit-setup/steps/useAddPasskeyFlow.ts:47)
   - **property** `cash.phone_already_registered.mode / reason`: draft lists mode and reason but the event only sends outcome; they come from other events' calls in the same file. Remove mode and reason from the draft; make outcome required. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:126)
   - **property** `cash.phone_submit_failed.mode / outcome`: draft lists mode and outcome but the event only sends reason (from the cash.phone_submitted and cash.phone_already_registered calls). Remove them from the draft; make reason required. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:78)
   - **property** `cash.phone_submitted.outcome / reason / source`: draft lists outcome, reason and source but the event only sends mode, from three call sites (useSubmitPhoneFlow.ts:68, verifyPhoneFlowStore.ts:106, useSubmitReviewFlow.ts:124). Remove the other three from the draft; make mode required. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitPhoneFlow.ts:68)
   - **property** `cash.phone_verified.reason / source`: draft lists reason and source but the event only sends mode. They belong to cash.phone_verify_failed and cash.kyc_awaiting_decision. Remove from the draft; make mode required. (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:91)
   - **property** `cash.phone_resend_failed.mode`: the draft marks reason as optional and mode as required; the only call (verifyPhoneFlowStore.ts:194) sends both every time, so both are required. The other scanner entries at :106 and :127 are calls of other events. (src/features/cash/stores/verifyPhoneFlowStore.ts:194)
   - **other** `cash.phone_verify_failed.reason`: codeInvalid in recovery is hard-coded reason client_error while an invalid code via the store path comes from the exception class; a wrong code is not distinguishable from other 4xx. Consider a distinct reason such as code_invalid (src/features/cash/screens/cash-deposit-setup/steps/useSubmitReviewFlow.ts:101)
   - **property** `cash.sign_in_*`: Scanner attached every sign-in call site to all four events, so each draft lists `reason` and `trigger` for all. Actually only cash.sign_in_failed sends reason; the other three send only trigger (src/features/cash/services/cashSignInService.ts:53)
   - **property** `Changed native currency input in Send flow / Changed token input in Send flow`: Drafts list screen, no_icon, no_price, total_tokens, which these events never send; they belong to token_list. Scanner attributed that call to them (src/features/transfer/screens/SendSheet.tsx:670)
   - **other** `cash.* events`: All cash.* names use a dot namespace, which does not match the snake_case pattern; the app uses this style for other namespaces too (perps., rewards.), so renaming needs a client-wide decision. Renames listed above use underscores (src/analytics/event.ts:132)
   - **property** `ENS events.failed_action/label`: The draft lists failed_action and label on the four ENS action events, but only the rap started/failed/completed events send them; these action events send only category. Remove them from the drafts (or add them to the calls). (src/features/ens/raps/actions.ts:394, 422, 428, 689-693)
   - **split status** `ENS rap events` → `ens_registration_completed`: ensRapFailed is a separate failure event, and the ENS action events fire on submission with no failure path, so a failed registration cannot be compared with a completed one. Put status + failure_reason on the outcome event. The failed event is not in this shard. (src/features/ens/raps/actions.ts:689)
   - **other** `discover.card_pressed / placement.interaction`: One tap sends both discover.card_pressed and a placement.interaction (card_press) with mostly the same properties; likewise carousel_scrolled and tab_pressed send surface.interaction. Pick one event per behaviour so counts aren't duplicated. (src/features/discover/utils/trackDiscoverCardPress.ts:28-52; src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17-27)
   - **merge** `Rejected new WalletConnect session / Approved new WalletConnect session / wc.failed_request`: Session approved, rejected and invalid-namespace failures are outcomes of the same proposal. Fold them into one outcome event with status = approved | rejected | failed and failure_reason. They are not in this shard. (src/features/wallet-connect/handlers/onSessionProposal.ts:62, 158, 175)
   - **other** `notification_permission.granted/denied`: If notificationPermissionRequested, granted and denied are not all in the plan, the request-to-answer funnel can't be measured. If handleEnable throws on the permission request, neither outcome fires, so the funnel will show a drop-off that isn't an abandon. (src/features/notifications/screens/NotificationPermissionScreen.tsx:63-75)
   - **property** `perps.* drafted properties`: The draft gave every perps event the union of keys from all calls in the same file, so properties the code does not send are listed. Not sent: errorMessage on opened_position, closed_position and added_to_position. positionSize, positionValue and entryPrice on added_to_position.failed. entryPrice, positionSize, positionValue, triggerOrders and errorMessage on open_position.canceled. Several optional ones on the .failed events. All are marked required:false with a note. Remove them when the draft is next regenerated. (src/features/perps/components/PerpsNavigatorFooter.tsx:237-272)
   - **privacy** `perps.trade_details.share_pressed.message`: The message property matches the never-send pattern ^message$. It holds the free-text result.message from the share library, which can name the app the person shared to. Drop it. success and shared already cover the outcome. (src/features/perps/screens/perps-trade-details-sheet/PerpsTradeDetailsSheet.tsx:81)
   - **property** `perps.trigger_order_canceled.errorMessage`: The draft lists errorMessage, but the success call at :60 doesn't send it. It comes from the failed call at :71, which uses perps.trigger_order_cancel.failed. Remove the property from this entry. triggerPrice, leverage and positionValue are sent on every call, so they are required. (src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:60)
   - **property** `perps.trigger_order_created.errorMessage`: The draft lists errorMessage, but the success call at :268 doesn't send it. It comes from the failure call at :281, which sends perps.trigger_order.failed. Remove the property here. triggerPrice, leverage and positionValue are sent on every call, so they are required. (src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:268)
   - **property** `perps.trigger_order.failed.triggerPrice`: The draft lists triggerPrice, leverage and positionValue, but the failure call at :281 doesn't send them; they come from the success call. Remove them from this entry. (src/features/perps/screens/CreateTriggerOrderBottomSheet.tsx:281)
   - **property** `perps.withdrew.errorMessage`: The draft lists errorMessage, but the success call at :48 sends only amount. The property belongs to perps.withdraw.failed. Remove it here. (src/features/perps/withdrawalConfig.ts:48)
   - **property** `predictions.place_order.orderId`: The draft lists orderId, reason, status and errorMessage, which place_order doesn't send; they come from the order_match.failed call at :353. Remove them here. (src/features/polymarket/utils/executePolymarketOrder.ts:324)
   - **property** `predictions.order_match.failed.orderAmountUsd`: The draft lists orderAmountUsd, feeAmountUsd, tokenAmount, spread, bestPriceUsd, orderPriceUsd and averagePriceUsd, which this event doesn't send; they come from the place_order call at :324. Remove them here. After the merge into place_order these amounts would be missing on failure rows. (src/features/polymarket/utils/executePolymarketOrder.ts:353)
   - **property** `predictions.redeem_position.transactionState`: The draft lists transactionState and errorMessage, which the success call at :59 doesn't send; they come from the failure call at :82. Remove them here. (src/features/polymarket/utils/redeemPosition.ts:59)
   - **property** `predictions.deposit.properties`: The draft has no properties because the call passes the handler's metadata object. The code sends amount (string), assetChainId (numeric ChainId), assetSymbol, executionStrategy (crosschainSwap | directTransfer | custom | swap) and sponsorship (sponsored | walletPaid), plus walletAddressHash and walletType. Add them. (src/features/polymarket/depositConfig.ts:60; src/systems/funding/types.ts:90)
   - **property** `predictions.deposit.failed.properties`: The draft has no properties. The code sends amount?, assetChainId?, assetSymbol?, error (string), stage (execution | prerequisite | validation | wallet), sponsorshipAttempted (boolean) and sponsorshipFailureReason? (insufficientSponsorBalance | sponsoredRelayExecutionFailed | unknownSponsorshipFailure), plus walletAddressHash and walletType. Add them. (src/features/polymarket/depositConfig.ts:59; src/systems/funding/types.ts:103)
   - **property** `predictions.withdrawal.properties`: The draft has no properties. The code sends amount (string) and targetChainId? (numeric ChainId, only for routed withdrawals), plus walletAddressHash and walletType. Add them. (src/features/polymarket/withdrawalConfig.ts:63; src/systems/funding/types.ts:829)
   - **property** `predictions.withdrawal.failed.properties`: The draft has no properties. The code sends amount?, error (string), stage (execution | prerequisite | validation) and targetChainId?, plus walletAddressHash and walletType. Add them. (src/features/polymarket/withdrawalConfig.ts:62; src/systems/funding/types.ts:839)
   - **property** `positions.error.properties`: The draft has no properties because the call passes a parsed object. The code sends kind (price_not_found | skipped_portfolio_item | unknown) and, by kind, tokenAddress and chainId (price_not_found), itemName and protocol (skipped_portfolio_item) or errorMessage (unknown). All of them except kind are conditional, so required: false. Add them. (src/features/positions/stores/trackPositionsResponseErrors.ts:9-23)
   - **other** `Rap started / Rap completed / Rap failed`: The scanner attributes the calls at actions.ts:394, 400, 422, 428 (Initiated ENS registration, Edited ENS records, Completed ENS registration, Extended ENS) to these three events. The real calls are at lines 689 (failed), 717 (started), 742 (completed). The sites at 394-460 send separate ENS events, which are not in this shard. (src/features/ens/raps/actions.ts:394)
   - **property** `request.approved / request.rejected`: Scanner attributes the call at SignTransactionSheet.tsx:295 to both request.approved and request.rejected; it only sends request.rejected, and approved is sent at lines 406 and 511. Draft required flags on approved were computed from a call that does not exist. (src/features/dapp-request/screens/SignTransactionSheet.tsx:295)
   - **property** `rnbw_airdrop.claim / rnbw_airdrop.claim.failed`: The draft gives both events the same properties because the scanner assigned all three calls (lines 49, 91, 116) to both. In code: success (line 91) sends chainId, claimId, claimedRnbw, claimedValueInCurrency, decimals, status, txHash, tenderlyUrl, durationMs, pollAttempts, platformRequestIds; failure (lines 49, 116) sends chainId, claimId, status, errorMessage, durationMs, platformRequestIds. Remove errorMessage from the success entry and the claimed*/decimals/pollAttempts/tx* properties from the failed one. (src/features/rnbw-rewards/utils/claimAirdrop.ts:91)
   - **other** `walletAddressHash / walletType`: Every draft lists these as required, but they come from analytics.track's default metadata, typed optional and undefined when wallet context isn't set (walletType may be undefined after wallet creation). They are marked required: false here. Consider listing them as super properties instead of per-event. (src/analytics/getWalletContext.ts:7)
   - **privacy** `rnbw_staking and rnbw_rewards failure events`: errorMessage forwards raw Error.message text from RPC, platform and wallet errors, which can contain addresses or request details. Replace it with a bounded failure_reason enum, or truncate and sanitise it. (src/features/rnbw-rewards/utils/claimRewards.ts:73,148; src/features/rnbw-staking/utils/stakeRnbw.ts:219; src/features/rnbw-staking/utils/unstakeRnbw.ts:77,123)
   - **property** `wc.failed_request.connector`: The draft lists connector, dappName and dappUrl, but no wc.failed_request call sends them. The scanner attributed onSessionProposal.ts:38, 62 and 158 to this event, but those lines send 'New WalletConnect pairing', 'Rejected new WalletConnect session' and 'Approved new WalletConnect session'. Remove the three properties from this entry. The draft also lacks type, reason and method, which the calls do send (type and reason are listed above). The scanner missed the calls at onSessionRequest.ts:84, 119, 147 and 214 and at NoNeedWCSheet.tsx:36. (src/features/wallet-connect/handlers/onSessionProposal.ts:38,62,158,175; src/features/wallet-connect/handlers/onSessionRequest.ts:84,119,147,214; src/screens/NoNeedWCSheet.tsx:36)
   - **property** `wc.failed_request.reason`: For `invalid namespaces`, reason carries the third-party validator's raw error message. It is unbounded and may include dapp-supplied strings. Replace it with a bounded failure_reason enum and drop the free text. (src/features/wallet-connect/handlers/onSessionProposal.ts:175)
   - **property** `wc.failed_request.type`: type values contain spaces ('read only wallet', 'method not supported') and the vocabulary is inconsistent with snake_case. Add `error_reason`/`failure_reason` snake_case values when renaming. Also `session_proposal` is allowed in the type union but never sent. (src/analytics/event.ts:662)
   - **add pair** `onSessionProposal approval failure`: The catch around approveSession (session approval failed) shows an alert but sends no analytics. A proposal that fails to approve is invisible, since Approved/Rejected/wc.failed_request don't fire. Consider a failure branch on the outcome event. (src/features/wallet-connect/handlers/onSessionProposal.ts:184)
   - **property** `app_store_review.prompted.promptCount`: The draft types it as string, but the code sends a number (promptTimestamps.length + 1; schema `promptCount: number`). Type it numeric. (src/utils/reviewAlert.ts:110)
   - **property** `app_store_review.prompted.walletAddressHash`: Not passed at the call site. The tracker adds walletAddressHash and walletType on every event as default metadata, and both can be undefined, so they are optional. Consider treating them as super properties rather than per-event properties. (src/analytics/index.ts:102)
   - **property** `buy_button.pressed.componentName`: The property says which button was pressed, so it plays the role of the shared entry_point. Rename it to entry_point and use the vocabulary values (for example eth_card, sheet_action_button), with a dual-fire window. (src/components/cards/EthCard.tsx:75; src/components/sheet/sheet-action-buttons/BuyActionButton.tsx:33)
   - **property** `chartType, candleResolution`: Property names are camelCase while the naming pattern is snake_case (chart_type, candle_resolution). This is app-wide, so decide it once for the whole plan rather than per event. (src/components/value-chart/Chart.tsx:272)
   - **privacy** `tappedDoneEditingWallet.wallet_label`: wallet_label is the wallet name the user typed, which is raw user text. Send a length (char_count) or drop it. This is raised here because the scanner attached it to this area. (src/screens/change-wallet/ChangeWalletSheet.tsx:264)
   - **property** `backup events: category and label`: category is always the constant 'backup' and carries no information; label holds the cloud platform and should be named provider. Drop category and rename label. (src/hooks/useWalletCloudBackup.ts:56)
   - **property** `Error during Google Drive Backup / Error during iCloud Backup / Error updating Backup status / iCloud not enabled / Ignore how to enable iCloud`: The drafts list error, label and category on all five events, because the scanner attached every call in the file to every event. In fact the Drive and iCloud errors send category, error, label; Error updating Backup status sends category, label; iCloud not enabled and Ignore how to enable iCloud send category only. The registry entries should match the actual calls. (src/hooks/useWalletCloudBackup.ts:56)
   - **privacy** `showWalletProfileModalForUnstoppableAddress / showWalletProfileModalForReadOnlyWallet / showWalletProfileModalForENSAddress`: These events send input, the raw text the user typed (an address, ENS or unstoppable domain name). watchWallet also sends addressOrEnsName and address. Wallet addresses are pseudonymous identifiers; the plan already uses walletAddressHash instead. Review whether the raw values should be sent. (src/hooks/useImportingWallet.ts:170)
   - **property** `mined_transaction.assets_resolved / mined_transaction.assets_timed_out`: The drafts list chainId, type and timeToResolve on both events, but resolved sends only timeToResolve and timed_out sends only chainId and type. The registry entries should match. (src/hooks/useWatchAssetUpdates.ts:70)
   - **privacy** `error_boundary.viewed.error`: The raw error object is sent as a property. Its message and stack can include user, wallet or request data. Send a bounded error name or type (e.g. error_name) instead, and keep the full detail in Sentry, which already holds it via eventId. (src/app/error-boundary/ErrorBoundary.tsx:21)
   - **privacy** `error_boundary.reset.error`: The raw error object is sent as a property. Send a bounded error name or type instead, and leave the detail in Sentry (eventId). (src/app/error-boundary/ErrorBoundary.tsx:14)
   - **property** `error_boundary.viewed.error`: The draft types error as string, but the code passes an unknown value (EventProperties declares error: unknown). The type is the draft's guess and the sent shape depends on PostHog serialization of an Error object. Fix this when the property is replaced. (src/analytics/event.ts:869)
   - **property** `f2c.provider.flow_completed.walletAddressHash`: walletAddressHash and walletType are added to every event by the analytics client (getDefaultMetadata), not by this call. They belong in _super_properties.yaml, not in the event's own property list. The scanner lists them because the client merges them in. (src/analytics/index.ts:145)
   - **property** `f2c.provider.flow_completed.success`: The draft typed success as string, but the code declares and sends a boolean. Correct the type. The value is sent only on the ramp branch and never as false, so a failed flow cannot be told apart from an unknown outcome. (src/handlers/deeplinks.ts:172)
   - **property** `f2c.provider.flow_completed.fiat_amount`: The event schema also allows fiat_amount, fiat_currency, fiat_source, crypto_network, crypto_amount, crypto_price, crypto_currency and crypto_fee, for providers like Ratio. None of these are sent by the calls the scanner found, so the draft does not list them. Add them only if a Ratio call exists. (src/analytics/event.ts:506)
   - **privacy** `swaps.searched_for_token.query`: raw user search text sent as an event property (matches never-send pattern ^query$); replace with char_count and, if useful, result_count (src/__swaps__/screens/Swap/components/TokenList/TokenToBuyList.tsx:174; src/__swaps__/screens/Swap/components/TokenList/TokenToSellList.tsx:89)
   - **privacy** `swaps.received_quote.quote`: the whole quote response object is sent (and whole asset objects in inputAsset/outputAsset); could carry wallet addresses or large payloads. Send a few flat fields (error_code, buy/sell amount, source) instead (src/__swaps__/screens/Swap/hooks/useSwapInputsController.ts:438-442)
   - **property** `swaps.changed_chain_id.inputAsset`: whole asset object sent as a property; reduce to flat values (input_asset_symbol, input_asset_chain_id) (src/__swaps__/screens/Swap/components/TokenList/ChainSelection.tsx:76)
   - **property** `performance.report.reportName`: The code sends reportName (currently only app_startup), and the draft lacks it. Add it as a string property. (src/performance/tracking/index.ts:184; src/analytics/event.ts:954)
   - **property** `performance.report.segments`: The code sends segments, an object of segment name to milliseconds. The draft lacks it. Add it as an object property. (src/performance/tracking/index.ts:171-191)
   - **property** `performance.report.data`: The code sends data, an object of free-form report params plus extra params. The draft lacks it. Add it as an object property. Its keys are not fixed by a schema, so enumerate them (initialRoute so far). (src/performance/tracking/index.ts:187-190; src/hooks/useApplicationSetup.ts:41)
   - **property** `Performance Wallet Initialize Time / performance.report`: The drafts type durationInMs as string, but the code sends a number (a millisecond difference). Correct to numeric. The same applies to performanceTrackingVersion, which is the number 3. (src/performance/tracking/index.ts:10,59,168)
   - **property** `performance.time_to_sign.error`: There is no status on this event: success and failure of the signing flow are told apart only by whether `error` is present. Consider adding status = success | failed and failure_reason so a conversion-style rate is queryable (rule 4). (src/state/performance/performance.ts:74)
   - **privacy** `performance.time_to_sign.error`: error is the raw error.message of an arbitrary exception. It could contain addresses, amounts or user data. Replace with an error code or a bucketed failure_reason. (src/state/performance/performance.ts:74)
   - **privacy** `wallet_initialization.failed.error`: error is the raw error.message of an arbitrary exception and may contain personal or wallet data. Replace with a bucketed failure_reason (e.g. keychain | import | unknown). (src/state/wallets/initializeWallet.ts:191)
   - **privacy** `performance.time_to_sign.operation.metadata`: metadata is free-form caller-supplied key/values. Check each caller to make sure it carries no addresses, amounts or other personal data. (src/state/performance/performance.ts:56)
   - **property** `performance.time_to_sign / performance.time_to_sign.operation`: The draft types for the numeric fields (completedAt, elapsedTime, totalElapsedTime, startTime, endTime, timeToCompletion) say string, but the code sends numbers. In the registry they should be numeric. Likewise resultWasNullish is boolean and metadata is object. (src/analytics/event.ts:797)
   - **privacy** `wallet_create.failed.error`: The catch branch sends the raw Error.message as a property. Its content is unbounded and may contain user or device data, and it makes the value impossible to group. Map it to an enum of reasons and log the raw message to the logger instead. (src/model/wallet.ts:730-733)
   - **property** `claimable-<type>-USDValue`: The code sends dynamic keys (`claimable-${type}-USDValue`, one per claimable type) with the claimables identify. The draft doesn't list them, and they break the naming pattern. Fold them into a fixed list of snake_case properties or an object property. (src/resources/addys/claimables/query.ts:73-83)
   - **property** `walletAddressHash / walletType`: analytics.identify() adds walletAddressHash and walletType to every identify call as default metadata. They are not in the draft. Add them (camelCase, so rename to wallet_address_hash and wallet_type), or document them as super properties. (src/analytics/index.ts:77-85; src/state/wallets/initializeWallet.ts:135)
   - **other** `set_once user properties`: branchCampaign, branchReferrer, branchReferringLink and the screen* properties are meant to be written once, and the code guards them itself (branchFirstReferringParamsSet, first-open block). analytics.identify has no set_once path, so the tool overwrites them as plain sets whenever the call runs. Use a real set_once call in the wrapper if first-value protection matters. (src/utils/branch.ts:115-135; src/App.tsx:190-199)
   - **property** `claimablesUSDValue / positionsUSDValue`: The same kind of value has two types: claimablesUSDValue is a string from add(), and positionsUSDValue is numeric. Claimables also sums amounts in the request currency while naming the result USD. Pick one type and confirm the currency. (src/resources/addys/claimables/query.ts:67-83; src/features/positions/stores/positionsStore.ts:169)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.
20. **user properties** (one PR)
   - **rename** `appIcon` → `app_icon`: camelCase breaks the snake_case pattern. Keep the old name until any dashboards using it move over. (src/hooks/useIdentifyAppIcon.ts:10)
   - **rename** `branchCampaign` → `branch_campaign`: camelCase breaks the snake_case pattern. (src/utils/branch.ts:127)
   - **rename** `branchReferrer` → `branch_referrer`: camelCase breaks the snake_case pattern. (src/utils/branch.ts:127)
   - **rename** `branchReferringLink` → `branch_referring_link`: camelCase breaks the snake_case pattern. (src/utils/branch.ts:127)
   - **rename** `candleResolution` → `candle_resolution`: camelCase breaks the snake_case pattern. (src/components/value-chart/Chart.tsx:282)
   - **rename** `chartType` → `chart_type`: camelCase breaks the snake_case pattern. (src/components/value-chart/Chart.tsx:271)
   - **rename** `claimablesAmount` → `claimables_amount`: camelCase breaks the snake_case pattern. (src/resources/addys/claimables/query.ts:83)
   - **rename** `claimablesUSDValue` → `claimables_usd_value`: camelCase breaks the snake_case pattern. (src/resources/addys/claimables/query.ts:83)
   - **rename** `currency` → `native_currency`: The pattern needs at least one underscore. A single word fails it, and 'currency' is ambiguous next to token currencies. (src/redux/settings.ts:115)
   - **rename** `enabledTestnets` → `enabled_testnets`: camelCase breaks the snake_case pattern. Its type is also boolean, while the draft says string. (src/redux/settings.ts:115)
   - **rename** `hardwareAccounts` → `hardware_accounts`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:66)
   - **rename** `hasImported` → `has_imported`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:74)
   - **rename** `importedPrivateKeys` → `imported_private_keys`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:71)
   - **rename** `importedRecoveryPhrases` → `imported_recovery_phrases`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:69)
   - **rename** `installSource` → `install_source`: camelCase breaks the snake_case pattern. (src/App.tsx:173)
   - **rename** `language` → `app_language`: The pattern needs at least one underscore. A single word fails it. (src/redux/settings.ts:153)
   - **rename** `ledgerDevices` → `ledger_devices`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:72)
   - **rename** `mostUsedNetworks` → `most_used_networks`: camelCase breaks the snake_case pattern. Its type is also array, while the draft says string. (src/features/network/stores/networkSwitcherStore.ts:49)
   - **rename** `notificationsPermissionStatus` → `notifications_permission_status`: camelCase breaks the snake_case pattern. (src/notifications/analytics.ts:24)
   - **rename** `numberOfImportedWalletsWithNotificationsTurnedOn` → `imported_wallets_notifications_on_count`: camelCase breaks the snake_case pattern, and the name is very long. (src/notifications/analytics.ts:119)
   - **rename** `numberOfWatchedWalletsWithNotificationsTurnedOn` → `watched_wallets_notifications_on_count`: camelCase breaks the snake_case pattern, and the name is very long. (src/notifications/analytics.ts:120)
   - **rename** `ownedAccounts` → `owned_accounts`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:65)
   - **rename** `positionsAmount` → `positions_amount`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:168)
   - **rename** `positionsAssetsAmount` → `positions_assets_amount`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:170)
   - **rename** `positionsDappsAmount` → `positions_dapps_amount`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:171)
   - **rename** `positionsRewardsAmount` → `positions_rewards_amount`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:172)
   - **rename** `positionsRewardsUSDValue` → `positions_rewards_usd_value`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:173)
   - **rename** `positionsUSDValue` → `positions_usd_value`: camelCase breaks the snake_case pattern. (src/features/positions/stores/positionsStore.ts:169)
   - **rename** `privateKeys` → `private_keys`: camelCase breaks the snake_case pattern. It is a count of wallets, not key material. (src/hooks/useWalletCohort.ts:70)
   - **rename** `recoveryPhrases` → `recovery_phrases`: camelCase breaks the snake_case pattern. It is a count of wallets, not phrase material. (src/hooks/useWalletCohort.ts:68)
   - **rename** `rnbwMembershipTier` → `rnbw_membership_tier`: camelCase breaks the snake_case pattern. (src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:43)
   - **rename** `screenHeight` → `screen_height`: camelCase breaks the snake_case pattern. (src/App.tsx:197)
   - **rename** `screenScale` → `screen_scale`: camelCase breaks the snake_case pattern. (src/App.tsx:197)
   - **rename** `screenWidth` → `screen_width`: camelCase breaks the snake_case pattern. (src/App.tsx:197)
   - **rename** `stakedRnbw` → `staked_rnbw`: camelCase breaks the snake_case pattern. (src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:42)
   - **rename** `trezorDevices` → `trezor_devices`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:73)
   - **rename** `watchedAccounts` → `watched_accounts`: camelCase breaks the snake_case pattern. (src/hooks/useWalletCohort.ts:67)
   - **remove** `trezorDevices`: The value is always 0: the wallet type switch never increments trezorDevices, so the property carries no information. Either count Trezor wallets or deprecate it. (src/hooks/useWalletCohort.ts:24-61)
   - **other** `enabledTestnets`: The property is written only at startup. Toggling testnets via settingsChangeTestnetsEnabled doesn't send identify, so profiles hold the value from the last launch. Add the identify call there. (src/redux/settings.ts:161-168)
   - Renamed events get deprecate + `replaced_by`, with a dual-fire window.

## Open questions

1. `add_favorite_token`: Does anyone ask about un-favoriting, or only about favorites being added? (src/resources/favorites.ts:174-176)
   Options: (A) Only adds are needed; (B) Track removals too
   Recommendation: Only adds are needed until someone asks about removals, as the budget gate requires.
2. `add_favorite_token.walletAddressHash`: What does `securelyHashWalletAddress` output look like (algorithm and length), so the example can be filled in? (src/analytics/getWalletContext.ts:27)
   Options: (A) Hex string; (B) Other
   Recommendation: Read src/analytics/utils.ts and give a real example of the format.
3. `add_favorite_token.address`: Is it right that the token address and name are fine to send as event properties, and that the PII check should be waived or the properties renamed? (src/resources/favorites.ts:178-183)
   Options: (A) Rename to token_address and token_name; (B) Keep the names and add a reviewed exception
   Recommendation: Rename to token_address and token_name.
4. `add_new_wallet_group.name.walletType`: Is walletType always sent for this event? The wallet context is set separately from the call and can be undefined right after wallet creation. (src/analytics/getWalletContext.ts:8)
   Options: (A) always set; (B) sometimes undefined
   Recommendation: sometimes undefined, since the type declares it optional
5. `add_new_wallet_group.name.walletAddressHash`: What does a walletAddressHash value look like? The code builds it with HMAC-SHA256 and a secret key, but there is no data snapshot to show the format. (src/analytics/utils.ts:62)
   Options: (A) hex string; (B) other
   Recommendation: check one real event in PostHog and write the format
6. `Tapped "Cancel" on Wallet Profile modal`: Does swiping the Wallet Profile sheet down or pressing the hardware back button also call handleCancel? If it doesn't, those dismissals are not tracked. (src/components/expanded-state/WalletProfileState.js:51)
   Options: (A) yes, all dismissals; (B) no, only the Cancel button
   Recommendation: no, only the Cancel button, until the ProfileModal code is checked
7. `token_details.nft`: Is token_details.nft used in a report? If it is, a dual-fire window is needed when renaming it. If not, rule 1 suggests removing it. (src/components/expanded-state/UniqueTokenExpandedState.tsx:389)
   Options: (A) on a dashboard; (B) unused
   Recommendation: ask the analytics owner before renaming
8. `add_new_wallet_group.name`: Does anyone ask a question this event answers? Nobody has stated one, and without the raw name the only measurable thing is the count of submits. (src/components/expanded-state/NewWalletGroupState.tsx:20)
   Options: (A) keep, with char_count; (B) remove
   Recommendation: keep with char_count only if wallet groups matter to a funnel; otherwise remove
9. `Changed language.language`: What values can language take (codes from supportedLanguages)? (src/screens/SettingsSheet/components/LanguageSection.tsx:11)
   Options: (A) en_US | es_419 | ...; (B) UNCONFIRMED
   Recommendation: Read the keys of supportedLanguages that have wallet translations and list them.
10. `Changed native currency.currency`: What are the NativeCurrencyKey values the app sends (fiat codes plus ETH and BTC)? (src/screens/SettingsSheet/components/CurrencySection.tsx:22)
   Options: (A) fiat codes only; (B) fiat codes | ETH | BTC
   Recommendation: List the keys of supportedCurrencies in src/features/currency/supportedCurrencies.
11. `claim_claimable.failed.claimableId`: Is claimableId meant to be the claimable's type (sponsored and transaction flows) or its unique id (airdrop flow)? (src/screens/Airdrops/utils.ts:124 and src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:76)
   Options: (A) claimable type; (B) unique id
   Recommendation: Use type everywhere, since the aggregate reports group by it, and add a separate claimable_unique_id if the id is needed.
12. `claim_claimable.succeeded`: Should an airdrop claim count as succeeded when the transaction is submitted, or only once it is confirmed on-chain? (src/screens/Airdrops/utils.ts:121)
   Options: (A) on submit (current); (B) after confirmation
   Recommendation: After confirmation, since onConfirm already waits for the receipt, or add a status 'pending' for the submitted state.
13. `claim_claimable.succeeded.claimableType`: Is multi_transaction a possible value of claimableType on succeeded and failed events? (src/screens/claimables/transaction/context/TransactionClaimableContext.tsx:470 and :418)
   Options: (A) only on succeeded (current schema); (B) both
   Recommendation: Both, if the transaction flow sends actionType for failures too. Failures currently hard-code 'transaction'.
14. `learn_article.opened.category`: What values can category, cardId and displayType take? (src/screens/LearnWebViewScreen.tsx:35)
   Options: (A) UNCONFIRMED
   Recommendation: Read the LearnCategory and CardType types and the Learn card keys, then list them.
15. `f2c.provider.flow_opened.sessionId`: Should sessionId also be passed to ramp, coinbase and stripe so outcomes can be joined? (src/screens/FiatOnRamp/ProviderListItem.tsx:30)
   Options: (A) Moonpay only (current); (B) all providers
   Recommendation: Pass it to every provider that supports a redirect URI.
16. `all events.walletAddressHash`: What is the format of walletAddressHash, and is it a salted hash that can't be reversed to an address? (src/analytics/getWalletContext.ts:27)
   Options: (A) salted hash; (B) unsalted hash
   Recommendation: Confirm how securelyHashWalletAddress works, then record an example.
17. `analytics_tracking.disabled`: Is it acceptable to send analytics_tracking.disabled before tracking stops, or should it be dropped for people who opt out? (src/screens/SettingsSheet/components/PrivacySection.tsx:35)
   Options: (A) keep (current); (B) remove
   Recommendation: Keep it only if privacy review agrees, because it is the one event sent after the person has said no.
18. `rewards.* and wallet events dashboards`: Are any of the dotted or title-case event names used on live dashboards, so the renames need a dual-fire window? (src/analytics/event.ts:125)
   Options: (A) none are on dashboards; (B) some are on dashboards
   Recommendation: Check dashboards before renaming; dual-fire for 30 days those that are used.
19. `spot_pnl.pill.shown.tokenAddress`: What do tokenAddress values look like (checksummed hex, lowercase, or a native-asset placeholder)? (src/screens/expandedAssetSheet/components/sections/BalanceSection.tsx:61)
   Options: (A) checksummed 0x address; (B) lowercase 0x address; (C) native asset placeholder possible
   Recommendation: Check a data sample once data arrives; no example is visible in the code.
20. `spot_pnl.pill.shown.chainId`: Is chainId sent as a number (as the event type says) while the draft says string? (src/analytics/event.ts:1319)
   Options: (A) numeric; (B) string
   Recommendation: numeric, per the event type in src/analytics/event.ts:1319.
21. `Tapped "Cancel" after editing wallet`: Should an empty name when the edit modal closes count as a cancel? Closing with a changed colour or avatar but no name also takes this branch. (src/screens/change-wallet/ChangeWalletSheet.tsx:263)
   Options: (A) keep as cancel; (B) rename to 'closed without name'
   Recommendation: Rename to a status of wallet_edit_completed so the meaning is explicit.
22. `Show Secret View.category`: Is 'settings backup' the only category ever sent, or do other views send this event with other values? (src/screens/SettingsSheet/components/Backups/ShowSecretView.tsx:11)
   Options: (A) settings backup only; (B) other categories exist
   Recommendation: settings backup only; the only call site in src sends this literal.
23. `rewards.viewed_sheet`: Should the view count every focus (current behaviour) or only the first open per visit? (src/screens/rewards/RewardsSheet.tsx:43)
   Options: (A) every focus; (B) first open per mount
   Recommendation: first open per mount, to match what 'viewed' means to a product owner.
24. `token_launcher.token_created`: Does TokenLauncherSDK.launchToken resolve when the transaction is submitted, or only after it is mined? That decides whether token_created means submitted or confirmed. (src/screens/token-launcher/state/tokenLauncherStore.ts:360-421)
   Options: (A) submitted (tx hash returned); (B) confirmed on-chain
   Recommendation: Treat it as submitted until the SDK is checked: the code fetches the transaction and adds it as pending right afterwards.
25. `token_launcher.creation_failed.operation`: Which values can operation take? They come from TokenLauncherSDKError.context and are not defined in this repo's code. (src/screens/token-launcher/state/tokenLauncherStore.ts:441)
   Options: (A) UNCONFIRMED: list from the SDK; (B) free text
   Recommendation: Read the error context types in the token launcher SDK and list the values as an enum.
26. `token_launcher.creation_failed.source`: Which values can source take (the same SDK error context)? (src/screens/token-launcher/state/tokenLauncherStore.ts:442)
   Options: (A) UNCONFIRMED: list from the SDK; (B) free text
   Recommendation: Read the SDK error context and list the values as an enum.
27. `Tapped Edit`: Does anyone use the edit-mode tap count in a report or dashboard? If not, it can be deprecated. (src/screens/change-wallet/ChangeWalletSheet.tsx:411)
   Options: (A) used, keep; (B) not used, deprecate
   Recommendation: not used, deprecate: it is a mode toggle with no decision attached.
28. `token_launcher.share_pressed`: Should share_pressed fire only when the user actually completes the share? Share.share's result action (sharedAction or dismissedAction) is ignored today. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:203-216)
   Options: (A) keep as is (sheet closed); (B) fire only on sharedAction
   Recommendation: fire only on sharedAction, or add a shared boolean property.
29. `Viewed profile`: Does anyone use profile views in a report or funnel? The effect refires when the ENS name loads or the params object changes, so counts are inflated, and the rules treat screen views that carry no decision as autocapture. (src/screens/ProfileSheet.tsx:66-74)
   Options: (A) Keep it, and fix the effect to fire once per open; (B) Remove it
   Recommendation: Remove it unless a report depends on it; otherwise fire once per sheet open
30. `Viewed profile.fromRoute`: Which route names can fromRoute take? (src/screens/ProfileSheet.tsx:70)
   Options: (A) UNCONFIRMED
   Recommendation: Grep the navigate calls to ProfileSheet and list the values
31. `top_holders.shown.chainId`: Which chain IDs can the top-holders section show? (src/screens/expandedAssetSheet/components/sections/TopHoldersSection.tsx:38-44)
   Options: (A) 1 only; (B) 1 | 8453 | other supported chains
   Recommendation: Take the list from the PnL-eligible token logic (usePnlEligibleToken)
32. `token_launcher.wallet_load_failed.error`: Is the 'Not authenticated' label correct whenever the key is truthy but not hex? Only kc.ErrorType.NotAuthenticated is checked, and any other non-hex value gets the same label. (src/screens/token-launcher/components/TokenLauncherFooter.tsx:73-81)
   Options: (A) Yes, only NotAuthenticated can occur; (B) No, split the cases
   Recommendation: Check the loadPrivateKey return values and split the labels if other values are possible
33. `BackupSheet shown.label`: Which values can the `type` prop of BackupSheetSection take? No caller was found under src/features/backup. (src/features/backup/components/BackupSheetSection.tsx:40,56)
   Options: (A) a fixed set of sheet types; (B) free text
   Recommendation: List the values from the callers' `type` props
34. `cash.buy_failed.failureReason`: Which OrderFailureReason values can the ramp backend return besides PaymentRejected? (src/features/cash/stores/cashBuyOrderStore.ts:145-146)
   Options: (A) PaymentRejected only; (B) PaymentRejected | other reasons
   Recommendation: Take the full list from the OrderFailureReason enum
35. `cash.buy_completed.network`: What values can the network and fiatCurrency of a completed order take? (src/features/cash/stores/cashBuyOrderStore.ts:125-127)
   Options: (A) one network and USD only; (B) several
   Recommendation: Take the values from the ramp client types
36. `walletAddressHash / walletType (all events)`: Can walletType be undefined on events sent right after wallet creation, and is walletAddressHash omitted then? (src/analytics/getWalletContext.ts:8-33)
   Options: (A) yes, both omitted; (B) no, always present
   Recommendation: Yes. getWalletContext returns {} without an address and walletType can be undefined.
37. `dappName / dappUrl`: Is it acceptable to send dapp-declared metadata (dappName, dappUrl) as event properties? (src/features/wallet-connect/handlers/onSessionProposal.ts:158-161)
   Options: (A) yes, public dapp metadata; (B) no
   Recommendation: Yes. It comes from the WalletConnect proposer metadata, not from the person.
38. `all cash.* events`: Is the dotted `cash.` prefix a deliberate namespace used across the app? The naming pattern doesn't allow dots, so each name is flagged for rename. (src/analytics/event.ts:126-155)
   Options: (A) Rename all to cash_* (snake_case); (B) Keep the dot and allow `.` in the naming pattern
   Recommendation: Rename to cash_* if nothing in the product analytics uses the dotted names yet, since no data is arriving.
39. `cash.card_linked.brand`: Which values can brand take? The code only shows 'Visa' as an example in a type comment. (src/analytics/event.ts:610-613; src/features/cash/stores/cardLinkFlowStore.ts:9,47)
   Options: (A) Visa | Mastercard | American Express | Discover; (B) UNCONFIRMED: check CardBrand in rampClient
   Recommendation: Check CardBrand in services/rampClient.ts and write the list into the example.
40. `walletAddressHash`: What does walletAddressHash look like (format and length) as sent by securelyHashWalletAddress? The example is UNCONFIRMED on every event because no value is visible in the code I was given. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) base64 string; (C) other
   Recommendation: Read src/analytics/utils securelyHashWalletAddress and put one sample in the registry; no data snapshot is available.
41. `cash.phone_verified.mode`: Can the verify-phone store ever send mode 'recovery' at verifyPhoneFlowStore.ts:127? The type allows it, but that branch calls finishSignupResume for any non-signup challenge, and recovery finishes in useSubmitReviewFlow.ts. (src/features/cash/stores/verifyPhoneFlowStore.ts:84-89)
   Options: (A) signup | resume (store) and recovery (review step); (B) signup | resume | recovery in both
   Recommendation: signup | resume | recovery, as the type allows, until someone confirms the store path.
42. `cash.phone_resend_failed`: Does anyone ask how often resending the code fails? If not, the event fails rule 1 and could be removed. (src/features/cash/stores/verifyPhoneFlowStore.ts:194)
   Options: (A) keep and add a resend success outcome; (B) remove
   Recommendation: keep and add a resend outcome only if support or growth looks at OTP delivery problems; otherwise remove.
43. `cash.wallet_check_failed`: Should a wallet-check outcome event exist (success | failed), or is a standalone failure event acceptable here? (src/features/cash/screens/add-cash-sheet/AddCashSheet.tsx:477)
   Options: (A) Add cash_wallet_check_completed with status; (B) Keep cash_wallet_check_failed standalone
   Recommendation: Keep standalone for now; the check is an internal step, and cash.amount_entered / the buy events already carry intent and outcome
44. `cash.wallet_link_failed`: Should a cancelled signature or passkey in the wallet-link flow be tracked, given it currently fires neither event? (src/features/cash/screens/add-wallet-sheet/useWalletLinkFlow.ts:42)
   Options: (A) Yes, status cancelled on cash_wallet_linked; (B) No, treat as abandonment (prompted with no outcome)
   Recommendation: Yes, status cancelled on the outcome, so abandonment from a cancelled passkey is distinguishable from not confirming
45. `walletAddressHash`: What exact format does the hash have (0x-prefixed hex?) and when is it undefined? (src/analytics/utils.ts:62)
   Options: (A) 0x + 64 hex chars; (B) 64 hex chars without prefix
   Recommendation: Confirm from a live event; computeHmac returns a 0x-prefixed hex string in ethers v5, and the value is undefined when no wallet is selected or the hash fails
46. `Clicked "Max" in Send flow input`: Does anyone need Max usage as its own event, or only as a property on the send outcome? (src/features/transfer/components/SendAssetFormField.js:71)
   Options: (A) Keep as event; (B) Replace with used_max property on sent_transaction; (C) Remove
   Recommendation: Replace with a used_max property on the send outcome
47. `walletAddressHash`: What is the format of walletAddressHash (algorithm and length), and is it safe to treat as non-identifying? (src/analytics/getWalletContext.ts:27 (securelyHashWalletAddress))
   Options: (A) Salted hash, not reversible; (B) Plain hash of the address
   Recommendation: Confirm in securelyHashWalletAddress and record an example value
48. `discover.carousel_scrolled`: Does anyone on the team ask whether carousels are scrolled? Scrolls are normally ignored by the design rules, and surface.interaction already records the same gesture. (src/features/discover/hooks/useTrackDiscoverCarouselScroll.ts:17-27)
   Options: (A) Keep it; (B) Remove it and rely on surface.interaction
   Recommendation: Remove it and rely on surface.interaction unless a product owner confirms the question
49. `discover.card_pressed.display / discover.section_pressed.display`: Which values can Display take (DISPLAY_VALUES)? (src/features/placements/surfaces/types.ts:9)
   Options: (A) Read from DISPLAY_VALUES
   Recommendation: Read DISPLAY_VALUES and list them in the examples
50. `discover.section_pressed.sectionId`: What are the section ids, since SectionId is a plain string and sections come from the surface configuration? (src/features/placements/surfaces/types.ts:13)
   Options: (A) Fixed set in the surface config; (B) Dynamic from the backend
   Recommendation: Ask the Discover owner for the list
51. `discover.prediction_order_pressed.outcome`: What values does outcome take (PolymarketMarket outcomes)? Is it a label such as Yes/No or an object? (src/features/discover/types/sectionLayout.ts:14)
   Options: (A) Yes | No; (B) Free-form outcome labels
   Recommendation: Check the PolymarketMarket type and record real examples
52. `New WalletConnect pairing.connector`: Which values can lastConnector take besides 'unknown'? (src/features/wallet-connect/handlers/onSessionProposal.ts:41)
   Options: (A) unknown only; (B) deeplink | qr | other
   Recommendation: Read services/pair.ts where lastConnector is set
53. `Didn't save backup password on iCloud`: Should the event carry a reason, so a user cancelling is distinguishable from a keychain error? (src/features/backup/backup.ts:534-536)
   Options: (A) Add failure_reason; (B) Leave it as is
   Recommendation: Add failure_reason when merging it into the saved-password event
54. `perps.*.market`: What do market symbols look like on the perps events (e.g. a bare ticker or a Hyperliquid-style id)? (src/features/perps/components/PerpsNavigatorFooter.tsx:250)
   Options: (A) bare ticker like BTC; (B) ticker with quote like BTC-USD; (C) other
   Recommendation: Check the first data after launch. Market is order.symbol or market.symbol from Hyperliquid.
55. `perps.*.errorMessage`: Which values can parseHyperliquidErrorMessage return? Is it a closed set of known messages, or can it pass through raw API text? (src/features/perps/components/PerpsNavigatorFooter.tsx:265)
   Options: (A) closed set; (B) raw API text
   Recommendation: Confirm before turning errorMessage into failure_reason, since only a closed list can be an enum.
56. `perps.open_position.canceled`: What makes createIsolatedMarginPosition resolve with nothing: the person rejecting a signature, an abort, or something else? (src/features/perps/components/PerpsNavigatorFooter.tsx:236)
   Options: (A) user rejected signing; (B) any early return in the action; (C) other
   Recommendation: Read hyperliquidAccountActions.createIsolatedMarginPosition. This decides whether the event can be called a cancel.
57. `pair_hw_wallet_nav.exited`: Is there an event that records a hardware wallet pairing actually succeeding? Does exited also fire after a successful pairing? (src/features/hardware-wallet/navigation/PairHardwareWalletNavigator.tsx:42)
   Options: (A) yes, a separate pairing-success event; (B) no, only exited
   Recommendation: If none exists, add a pairing outcome with status, so abandon vs. success is measurable.
58. `walletAddressHash`: What does the hash format look like (length, encoding) for securelyHashWalletAddress? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) other
   Recommendation: Take a real value from the first events to fill example.
59. `renamed events`: Which of the dotted or title-case events here (notification_permission.*, pair_hw_wallet_nav.*, perps.*, Opened position Sheet, Opened asset from position) are used in dashboards, funnels or cohorts? (src/analytics/event.ts:95-331)
   Options: (A) none; (B) some (list them)
   Recommendation: Dual-fire the ones on a dashboard for 30 days. Rename the rest straight away.
60. `Opened asset from position.assetValue`: Is position.totals.total.amount always in USD, or is it already in the native currency? (src/features/positions/screens/PositionSheet.tsx:90-92)
   Options: (A) always USD; (B) native currency
   Recommendation: If it is USD, send positionsCurrency and assetCurrency as USD, or convert the amounts.
61. `perps.* market, perpsBalance, triggerPrice examples`: What do real values look like for market (Hyperliquid symbol, e.g. a bare coin name or with a quote suffix), perpsBalance and amount (USD with decimals?), and what format does walletAddressHash take? No code literal shows them and there is no data snapshot. (src/features/perps/screens/perp-detail-screen/TriggerOrdersSection.tsx:61)
   Options: (A) market is the bare Hyperliquid coin symbol; (B) market includes a quote suffix
   Recommendation: Check one captured event per type and paste the real value into the examples.
62. `predictions.* eventSlug, marketSlug, outcome, tokenId, transactionState`: What values do outcome, eventSlug, marketSlug, tokenId and the relayer transactionState take in practice? (src/features/polymarket/utils/executePolymarketOrder.ts:324; src/features/polymarket/utils/redeemPosition.ts:59)
   Options: (A) outcome is Yes | No only; (B) outcome also holds team or candidate names
   Recommendation: Take example values from a real captured order and redeem; outcome is likely the Polymarket outcome label, so it is not a closed enum.
63. `predictions.deposit`: Is predictions.deposit meant to mean 'deposit submitted' or 'deposit confirmed on-chain'? The code fires it when the executor returns success, while isConfirmed and waitForConfirmation are only passed to the post-submit handler. (src/systems/funding/hooks/useDepositHandler.ts:125-144)
   Options: (A) submitted; (B) confirmed on-chain
   Recommendation: submitted; name it in the description and add a confirmed event only if someone needs it.
64. `errorMessage properties`: Can the error texts sent as errorMessage (Hyperliquid, Polymarket and relayer errors, plus the raw backend string for positions.error kind unknown) contain wallet addresses or other personal data? Raw text is on the never-send list. (src/features/perps/withdrawalConfig.ts:54-60; src/features/positions/stores/trackPositionsResponseErrors.ts:22)
   Options: (A) No, they are SDK or backend error codes; (B) Sometimes; map to a short failure_reason enum
   Recommendation: Map to a failure_reason enum when folding the _failed events into the outcome events, and keep the raw text out of analytics.
65. `dotted event names`: Are any of these dotted event names used in dashboards, funnels or cohorts? That decides whether the renames need a dual-fire window or can be a straight cut-over. (src/analytics/event.ts:323-349)
   Options: (A) Yes, dual-fire 30 days; (B) No, rename directly
   Recommendation: Ask the analytics owner for the dashboard list before scheduling the rename. The dot style is used across event.ts, so one decision should cover all of them.
66. `walletAddressHash`: What does securelyHashWalletAddress return (algorithm, length, any salt)? An example value is needed for the registry. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string (UNCONFIRMED length); (B) other
   Recommendation: Document the format from securelyHashWalletAddress and put one sample value as the example.
67. `Rap started.label`: Which values can the rap name (getRapFullName) take, and which ENSRapActionType values can failed_action take? (src/features/ens/raps/actions.ts:715)
   Options: (A) list the ENS rap names and action types; (B) leave as free-form string
   Recommendation: List them as an enum, from createENSRapByType and the ENSRapActionType definition.
68. `request.approved.network`: Which chain-name values can network take (chainsName[chainId])? (src/features/dapp-request/screens/SignTransactionSheet.tsx:412)
   Options: (A) names from the backend networks store, e.g. mainnet | base | optimism; (B) free-form
   Recommendation: Take the example list from useBackendNetworksStore.getChainsName() and mark it open-ended.
69. `request.rejected`: Do you want user cancellations and signing/sending failures kept apart in the dapp-request outcome? (src/features/dapp-request/screens/SignTransactionSheet.tsx:523)
   Options: (A) keep one rejected event; (B) status = success | cancelled | failed on one outcome event
   Recommendation: One outcome event with status success | cancelled | failed and failure_reason.
70. `Rejected new WalletConnect session`: Which other code paths call rejectProposal, and with which reasons? (src/features/wallet-connect/handlers/onSessionProposal.ts:45)
   Options: (A) only invalid namespaces; (B) several reasons (list them)
   Recommendation: Grep callers of rejectProposal; add a reason property if there is more than one.
71. `rnbw_rewards.claim.status`: Does the rewards claim status use the same enum as the airdrop one, and can UNSPECIFIED reach the success event? (src/features/rnbw-rewards/utils/pollClaimStatus.ts:45)
   Options: (A) same enum; success is CLAIM_STATUS_CONFIRMED only; (B) other values possible
   Recommendation: Check CLAIM_PENDING_STATUSES and rewardsClaimSchemas.claimStatus; document the exact values that can come through.
72. `rnbw_airdrop.claim.failed.errorMessage`: Is raw exception text acceptable as errorMessage, or should it be mapped to a fixed failure_reason list? (src/features/rnbw-rewards/utils/claimAirdrop.ts:113)
   Options: (A) keep errorMessage; (B) map to a fixed failure_reason enum
   Recommendation: Map to a fixed failure_reason enum (prepare | sign | api | poll_timeout | claim_failed | unknown) and drop raw text.
73. `rnbw_rewards.claim.failed.status`: What values can the backend claim status take (e.g. PENDING, COMPLETED, FAILED)? (src/features/rnbw-rewards/utils/claimRewards.ts:149)
   Options: (A) PENDING | COMPLETED | FAILED; (B) UNCONFIRMED
   Recommendation: Confirm with the platform API schema (GetClaimIntent / ClaimRewards response). Tests show PENDING and FAILED exist for staking executions, but the claim enum isn't visible.
74. `Updated Gas Price.gasPriceOption`: What are the exact string values of the gas option constants (NORMAL, FAST, URGENT, CUSTOM)? (src/features/gas/redux/gas.ts:481,488)
   Options: (A) normal | fast | urgent | custom; (B) UNCONFIRMED
   Recommendation: Read the constants in the gas feature and fill the example from them.
75. `Set App Icon`: Does anyone ask which app icons users pick? The `appIcon` user property already holds the current icon. (src/features/app-icon/components/AppIconSettingsSection.tsx:32)
   Options: (A) Keep, for icon adoption; (B) Remove; use the user property
   Recommendation: Remove if appIcon is already set as a user property (rule 1 and the settings-tweak exclusion).
76. `surface.interaction.display / destination / sectionId`: What values do SurfaceId, Display, Destination and SectionId take? They are string unions in placements/surfaces/types, which I didn't read. (src/analytics/event.ts:1013-1020)
   Options: (A) Read the type definitions; (B) Provide the list
   Recommendation: Fill the examples from placements/surfaces/types.ts.
77. `Updated Gas Price`: Should this event be tied to swaps, or does it fire from every flow that changes the gas option (send, ENS, swaps)? If shared, add a `flow` or entry_point property. (src/features/gas/redux/gas.ts:476-489)
   Options: (A) Swaps only; (B) All flows; add entry_point
   Recommendation: Add entry_point: callers of gasUpdateGasFeeOption were not inspected in this shard.
78. `Sent transaction.network / chainId / assetAmount examples`: What do `network` and `assetName` look like in practice, and what number format does toAnalyticsAmount return? (src/features/transfer/utils/trackSentTransaction.ts:35,37)
   Options: (A) Provide sample values; (B) Read toAnalyticsAmount
   Recommendation: Read src/analytics/utils.ts (toAnalyticsAmount) to confirm numeric output; take network values from the backend network names.
79. `walletAddressHash / walletType (all events)`: What are the allowed values of walletType and the hash format of walletAddressHash? They are added in src/analytics/index.ts from getWalletContext. (src/analytics/index.ts:20-23)
   Options: (A) Read getWalletContext; (B) Provide values
   Recommendation: Read src/analytics/getWalletContext.ts and fill the example.
80. `wc.failed_request.method`: Which RPC methods show up in `method` for unsupported-method failures? (src/features/wallet-connect/handlers/onSessionRequest.ts:217)
   Options: (A) any dapp-sent method string; (B) fixed list
   Recommendation: Confirm from data once available; the code passes the dapp's raw method string.
81. `wc.failed_request.walletAddressHash`: What does the hash look like (format and algorithm)? I couldn't see securelyHashWalletAddress. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex digest; (B) other
   Recommendation: Read src/analytics/utils to give an example.
82. `wc.failed_request.type`: Is `session_proposal` still intended as a type value? No call sends it. (src/analytics/event.ts:662)
   Options: (A) remove from type union; (B) keep, call is planned
   Recommendation: remove from type union
83. `app_store_review.prompted.walletAddressHash`: What does securelyHashWalletAddress produce, so the example can show the real format? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex digest; (B) other
   Recommendation: Read src/analytics/utils securelyHashWalletAddress and write the real format as the example.
84. `card.pressed.cardType`: Which values can LearnCard's type (CardType) take besides stretch and square? (src/components/cards/LearnCard.tsx:31-35)
   Options: (A) stretch | square; (B) stretch | square | other
   Recommendation: stretch | square, if CardType in GenericCard is only those two
85. `buy_button.pressed.routeName`: What route names can routeName take on EthCard and BuyActionButton? The code only reads them from useRoute() at runtime. (src/components/cards/EthCard.tsx:57; src/components/sheet/sheet-action-buttons/BuyActionButton.tsx:21)
   Options: (A) List the route names from Routes; (B) Leave open
   Recommendation: List them from the navigator's route names once confirmed
86. `buy_button.pressed`: Does navigateToAddCash always invoke its callback? The event only fires inside it, so any path that skips the callback also skips the event. (src/components/cards/EthCard.tsx:67-78)
   Options: (A) Always calls it; (B) Sometimes skips it
   Recommendation: Check useAddCashRoute and confirm before relying on press counts
87. `qr_code.viewed`: Is there a separate event when the RECEIVE_MODAL is shown, or is the tap on the card the only signal? (src/components/cards/ReceiveAssetsCard.tsx:46-51)
   Options: (A) The tap is enough; (B) Add a view event in the modal
   Recommendation: The tap is enough until someone asks about modal views
88. `Changed Chart Type / Changed Candle Resolution`: Does the useListen hook also fire its callback on the first mount, with the persisted value? If it does, each event also fires once per mount without a user action. (src/components/value-chart/Chart.tsx:266)
   Options: (A) Only on actual changes; (B) Also on mount with the persisted value
   Recommendation: Check useListen in the @storesjs/stores package; the description assumes it fires only on changes.
89. `Changed Chart Type.walletAddressHash`: What does walletAddressHash look like (hash algorithm and length)? The example could not be derived from the code I was given. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) other
   Recommendation: Read securelyHashWalletAddress in src/analytics/utils and put one sample format in the example.
90. `Changed Global Notification Settings.topic`: Which global notification topics exist or are planned? `GlobalNotificationTopic` is empty in the code, so no values can be listed. (src/notifications/settings/constants.ts:11)
   Options: (A) None exist, remove the event; (B) Topics are added in an upcoming release
   Recommendation: If no topics are planned, deprecate the event. It can't fire from the current Settings UI.
91. `Tapped Push Notification`: Are the legacy event names ('Tapped Push Notification', 'Changed Global Notification Settings') used in dashboards or funnels? This decides whether a dual-fire window is needed for the renames. (src/analytics/event.ts:100-101)
   Options: (A) Yes, dual-fire for 30 days; (B) No, rename directly
   Recommendation: Check PostHog dashboards and insights before renaming.
92. `Tapped Push Notification.campaign`: Does the push payload `data.type` ever carry values other than transaction, wc and marketing, e.g. a real campaign id from the backend? (src/notifications/types.ts:3)
   Options: (A) transaction | wc | marketing | default only; (B) Other values are sent by the backend
   Recommendation: Treat it as transaction | wc | marketing | default, which are the values the app handles in `NotificationTypes`, until the backend team confirms.
93. `claimable_panel.opened.usdValue`: What is the type of claimable.totalCurrencyValue.amount (string or number), and is it in USD or the user's selected currency? (src/components/asset-list/RecyclerAssetList2/Claimable.tsx:97)
   Options: (A) string, USD; (B) string, user currency; (C) numeric, USD
   Recommendation: Check the ClaimableType definition. If the value follows the user's currency, rename the property to something like currency_value.
94. `claimable_panel.opened.claimableId`: What values can claimable.type take? (src/components/asset-list/RecyclerAssetList2/Claimable.tsx:90)
   Options: (A) a fixed list of claimable programs; (B) free-form IDs per claimable
   Recommendation: List the values from the ClaimableType definition so the example can be a real enum.
95. `claimable_panel.opened`: Is there an outcome event for claiming (success, failure or cancelled), so that opening the panel has a matching end? (src/components/asset-list/RecyclerAssetList2/Claimable.tsx:99)
   Options: (A) yes, in the claim panel; (B) no, only the open is tracked
   Recommendation: If none exists, add one with status and failure_reason (rule 2 and rule 4). The claim panel is outside this shard.
96. `walletAddressHash`: What is the format of walletAddressHash (the algorithm and length from securelyHashWalletAddress)? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex digest; (B) UNCONFIRMED
   Recommendation: Read src/analytics/utils securelyHashWalletAddress and put a real sample in the example.
97. `Pressed Button`: Are there other call sites for Pressed Button outside the asset list and change-wallet screens, so that the buttonName and action lists are complete? (src/analytics/event.ts:81)
   Options: (A) only these two; (B) more elsewhere
   Recommendation: Search for analytics.event.pressedButton across the repo before turning buttonName into an enum.
98. `debug.identify_probe.value`: Which currency values can the currency probe send (the String of nativeCurrency)? (src/redux/settings.ts:121)
   Options: (A) ISO codes such as USD | EUR; (B) other
   Recommendation: Check the native currency list in the settings code and fill the example from it
99. `debug.identify_probe`: Is FEPLAT-67 finished, so the probe can be removed now? (src/App.tsx:174-176)
   Options: (A) yes, remove; (B) no, keep until the ticket closes
   Recommendation: Remove if the ticket is closed
100. `First App Open.walletAddressHash`: Can walletAddressHash or walletType ever be set on first open, before any wallet exists? (src/analytics/getWalletContext.ts:22)
   Options: (A) no, always empty; (B) yes, in some flows
   Recommendation: No: the wallet is created after the first open, so drop these two properties from this entry
101. `Error during iCloud Backup.label`: What value does cloudPlatform take on iOS? Only the 'Google Drive' string is visible in the code. (src/hooks/useWalletCloudBackup.ts:132)
   Options: (A) iCloud; (B) iCloud Drive; (C) other
   Recommendation: iCloud, but confirm where cloudPlatform is defined in useWalletCloudBackup.ts
102. `pending_transaction.resolved.type`: Which values can a transaction type take, so that the examples for type on the pending_transaction and mined_transaction events can be filled in? (src/hooks/useWatchPendingTxs.ts:92)
   Options: (A) send | swap | bridge | approve | other (list from the TransactionType enum)
   Recommendation: Read the transaction type enum and list the full set
103. `pending_transaction.resolved.chainId`: Which chain ids can occur? The mainnet value 1 is the only one seen in the code. (src/hooks/useWatchPendingTxs.ts:92)
   Options: (A) all backend-supported networks; (B) a fixed list
   Recommendation: all backend-supported networks; write the example as the current list
104. `walletAddressHash`: What is the format of walletAddressHash (securelyHashWalletAddress)? Is it salted and can it be reversed, so is it safe to treat as a non-PII identifier? (src/analytics/getWalletContext.ts:27)
   Options: (A) salted hash, not reversible; (B) unsalted hash
   Recommendation: Confirm in src/analytics/utils
105. `Imported seed phrase`: Should the import success event be kept as a seed-phrase-only event, or cover all imports (key, address, hardware) with an import_type property? (src/hooks/useImportingWallet.ts:337)
   Options: (A) seed phrase only; (B) all imports with import_type
   Recommendation: all imports with import_type; the code path is shared
106. `Show wallet profile modal for imported wallet.type`: Which values can walletResult.type take? (src/hooks/useImportingWallet.ts:279-282)
   Options: (A) seed phrase | private key; (B) UNCONFIRMED: read from deriveAccountFromWalletInput
   Recommendation: Read the enum from deriveAccountFromWalletInput and list the values
107. `walletAddressHash`: Is the hash always the 0x-prefixed HMAC-SHA256 hex string, and is it OK that it describes the active wallet, not the one being imported or watched? (src/analytics/utils.ts:62-83)
   Options: (A) Yes, always 0x<64 hex>; (B) No, the format differs
   Recommendation: Confirm the format with the analytics owner and document it in _properties.yaml; the active-wallet semantics matter for the import events
108. `Viewed ENS profile.ens`: Which analytics questions need the ENS name itself rather than a flag? (src/hooks/useOnAvatarPress.ts:111-115)
   Options: (A) Keep ens; (B) Replace it with has_ens boolean
   Recommendation: Replace it with has_ens unless someone needs the name
109. `Scanned Rainbow profile url`: Should the QR scan events be tracked as the intent plus an outcome (profile opened or failed)? Today the event fires before validation. (src/hooks/useScanner.ts:78-97)
   Options: (A) Keep it as detection only; (B) Add a status property for the outcome
   Recommendation: Merge into qr_scanned with scan_type, and keep a status for the outcome only if someone asks
110. `watch_wallet.walletType`: Which `walletType` value should a watch_wallet event carry: the active wallet's type, or `watched` for the wallet being added? (src/analytics/index.ts:147-148; src/analytics/getWalletContext.ts:8-31)
   Options: (A) active wallet's type (as now); (B) always watched
   Recommendation: Add an explicit property for the wallet being watched and keep `walletType` as the active wallet's type.
111. `watch_wallet.walletAddressHash`: What does `securelyHashWalletAddress` produce, and is the hash salted? The example is UNCONFIRMED because no data snapshot was provided. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex digest; (B) other
   Recommendation: Confirm the format against a received event.
112. `watch_wallet`: Where is `handlePressImportButton` called with `type: 'watch'`? Is it only from the Watch an address screen, or also from deep links or other entry points? (src/hooks/useImportingWallet.ts:118-129)
   Options: (A) one screen only; (B) several entry points
   Recommendation: If there are several, add an `entry_point` property.
113. `error_boundary.viewed.eventId`: What does a real eventId look like? Is it always the Sentry event ID, and how often is it empty? (src/app/error-boundary/ErrorBoundary.tsx:20-26)
   Options: (A) Sentry event ID (32-char hex), '' when none; (B) Other format
   Recommendation: Sentry event ID, with '' as the empty fallback. Confirm against a real event.
114. `error_boundary.viewed.error`: What do these events actually carry in the error property? Is it a serialized Error (message and stack), an empty object, or a string? (src/app/error-boundary/ErrorBoundary.tsx:14-25)
   Options: (A) Error message string; (B) Empty object {}; (C) Serialized message and stack
   Recommendation: Check a live event. If it holds messages, replace it with a bounded error_name, per the privacy item.
115. `error_boundary.viewed.walletAddressHash`: What is the format of walletAddressHash (hash algorithm and length)? Is it safe as a person-level identifier in analytics? (src/analytics/getWalletContext.ts:27)
   Options: (A) Hex digest, fixed length; (B) Truncated digest
   Recommendation: Read securelyHashWalletAddress in src/analytics/utils and put one example value in the registry.
116. `f2c.provider.flow_completed`: Is there a Ratio (or other SDK-callback) call to this event outside src/handlers/deeplinks.ts? The FiatProviderName enum has no Ratio value. (src/handlers/deeplinks.ts:160; src/entities/f2c.ts:1)
   Options: (A) No, only the deeplink calls exist; (B) Yes, in a file the scan missed
   Recommendation: Check with the engineer who owns the on-ramp. The comment at src/handlers/deeplinks.ts:160 says one exists.
117. `f2c.provider.flow_completed.walletAddressHash`: What does a sample walletAddressHash look like (hash algorithm and length)? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) UNCONFIRMED
   Recommendation: Read securelyHashWalletAddress in src/analytics/utils, or take a value from the data once it arrives.
118. `f2c.provider.flow_completed.sessionId`: Should `sessionId` be marked required? Both deeplink calls always send it, but the event schema declares it optional. (src/handlers/deeplinks.ts:151)
   Options: (A) required; (B) optional
   Recommendation: Required: both calls bail out before tracking when it is missing.
119. `swaps.submitted`: After errorMessage === 'handled' (and the sponsorship branch), does the code in getNonceAndPerformSwap really fall through to also fire swaps.submitted, so one swap yields both swaps.failed and swaps.submitted? (src/__swaps__/screens/Swap/providers/swap-provider.tsx:337-400)
   Options: (A) yes, a bug to fix; (B) no, 'handled' returns earlier
   Recommendation: Check with an engineer; the code at lines 349-362 only returns for non-'handled' errors, so 'handled' continues to line 396
120. `swaps.failed.errorMessage`: What values can errorMessage take besides the sentinel 'handled'? Does the text ever include user data (addresses, amounts)? (src/__swaps__/screens/Swap/providers/swap-provider.tsx:337-360)
   Options: (A) a fixed set of error strings; (B) free text from RPC / relayer errors
   Recommendation: Treat as free text; bucket into a failure_reason enum
121. `swaps.failed.tradeAmountUSD`: What type is quote.tradeAmountUSD (number or numeric string)? (src/__swaps__/utils/trackSwapEvent.ts:67)
   Options: (A) number; (B) string
   Recommendation: number
122. `walletAddressHash`: What does the hash from securelyHashWalletAddress look like (algorithm, length)? Needed for a real example. (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) UNCONFIRMED
   Recommendation: Read src/analytics/utils and put one example in the shared vocabulary
123. `swaps.flipped_assets.inputAmount`: Is inputAmount a string or a number? Event typing allows both. (src/__swaps__/screens/Swap/components/FlipButton.tsx:113)
   Options: (A) string; (B) numeric
   Recommendation: Check SwapInputController.inputValues.inputAmount; likely numeric string
124. `insufficient_native_asset_for_action.type`: Besides 'Swap', which route names reach useTrackInsufficientGas (send sheet, other flows)? (src/features/gas/hooks/useGas.ts:83,176)
   Options: (A) Swap | SendSheet | others; (B) Swap only
   Recommendation: List the callers of useGas and enumerate their route names
125. `insufficient_native_asset_for_action`: Is the call in reportInsufficientFunds (SyncSwapStateAndSharedValues) still reached? No caller was seen in the code provided, and the swap screen may also use useTrackInsufficientGas, which would double-fire. (src/__swaps__/screens/Swap/providers/SyncSwapStateAndSharedValues.tsx:110-115)
   Options: (A) live; (B) dead code, remove
   Recommendation: Grep for reportInsufficientFunds callers; remove if none
126. `performance.report.data`: Which keys can data carry besides initialRoute, and which values does initialRoute take? (src/hooks/useApplicationSetup.ts:41; src/App.tsx:162)
   Options: (A) initialRoute only; (B) initialRoute plus extras passed to finishReport
   Recommendation: initialRoute only, since the only addReportParams call and the only finishReport call found pass no other keys
127. `performance.report.segments`: load_js_bundle and load_main_module are defined as report segments, but no call to log them was found in the code searched. Are they logged from native code, or unused? (src/performance/tracking/index.ts:21-22)
   Options: (A) logged from native code; (B) unused
   Recommendation: Confirm with the mobile team before listing them as segment keys
128. `Performance Wallet Initialize Time.walletAddressHash`: What is the format of walletAddressHash (hash algorithm and length), so a real example can be written? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex digest string; (B) other
   Recommendation: Check securelyHashWalletAddress and fill the example from it
129. `performance.report.durationInMs`: Are either of the performance events used on a dashboard or alert, which would require a dual-fire window for the rename? (src/analytics/event.ts:246-247)
   Options: (A) yes, on a dashboard; (B) no
   Recommendation: Ask the analytics owner before renaming. If neither is used, rename without dual-fire.
130. `performance.time_to_sign.elapsedTime`: Are elapsedTime (sum of timed steps) and totalElapsedTime (wall clock from flow start) both used in a report? Does anyone need both? (src/state/performance/performance.ts:72)
   Options: (A) both are used; (B) only totalElapsedTime; (C) only elapsedTime
   Recommendation: keep both until someone confirms; they differ by the time spent between timed steps (e.g. the person reading the sheet)
131. `performance.time_to_sign.operation`: Does anyone analyse the per-step breakdown, or only the flow total? The operation event fires for every wrapped step, which is much more volume than time_to_sign. (src/state/performance/performance.ts:62)
   Options: (A) per-step is queried; (B) only the total is queried
   Recommendation: keep it if step-level latency is monitored; otherwise propose removing it
132. `wallet_initialization.failed.error`: Which error messages does initializeWallet actually throw, so a failure_reason enum can replace the raw text? (src/state/wallets/initializeWallet.ts:190)
   Options: (A) keychain errors only; (B) keychain | import | network | unknown
   Recommendation: read the error classes thrown by walletInit and map them to a short enum
133. `performance.time_to_sign.walletType`: The draft says walletType could be 'hardware', but tracking is disabled for hardware wallets. Is that intended, so that hardware-wallet signing time is never measured? (src/state/performance/performance.ts:30)
   Options: (A) intended; (B) hardware wallets should be measured too
   Recommendation: intended, confirm with the owner of the performance module
134. `Search Query`: Is firing only on clear intended? Searches that end by Cancel or by navigating away are not tracked. (src/components/Discover/DiscoverSearchInput.tsx:132-142)
   Options: (A) Yes, only on clear; (B) Also track when search settles or is cancelled
   Recommendation: Track a settled search with char_count and result_count instead of on clear
135. `Time spent on the Discover screen.durationInMs`: Should time spent while the app is backgrounded on Discover be excluded from the duration? (src/components/Discover/useTrackDiscoverScreenTime.ts:14-27)
   Options: (A) Include (current behaviour); (B) Exclude
   Recommendation: Exclude, if the metric is meant to mean active time
136. `walletAddressHash`: What does securelyHashWalletAddress produce (hash algorithm and format), so the example can be written? (src/analytics/getWalletContext.ts:27)
   Options: (A) hex string; (B) other
   Recommendation: Read src/analytics/utils and fill in the example
137. `wallet_create.failed.walletAddressHash`: What format does securelyHashWalletAddress return, such as hash algorithm and length? Is the wallet context already set for the new wallet when createWallet fires these events? (src/analytics/getWalletContext.ts:24-31, src/analytics/index.ts:145-149)
   Recommendation: Confirm with an engineer. The draft example stays UNCONFIRMED until then.
138. `wallet_create.succeeded.walletLibraryType`: Which values can walletLibraryType take? The WalletLibraryType type definition wasn't in the code I was given. (src/analytics/event.ts:403)
   Options: (A) UNCONFIRMED
   Recommendation: Read the WalletLibraryType definition when adding the property to the registry.
139. `wallet_create.failed.error`: Which exception messages does the catch branch send in practice? Can the list be collapsed into a few failure_reason values? (src/model/wallet.ts:730)
   Options: (A) Keep raw messages; (B) Map to an enum such as keychain_error | derivation_error | unknown
   Recommendation: Map to an enum, with the three fixed codes kept as they are.
140. `identity.reset`: Should the profile be reset when the user wipes the app or removes all wallets? The code never calls reset, and the id is a persistent device id. (src/App.tsx:168-173)
   Options: (A) No reset: device id is permanent; (B) Reset on wallet wipe or logout
   Recommendation: No reset, since the id is a keychain device id and there are no accounts to log out of. Confirm with the owner.
141. `identity.alias`: Is skipping alias deliberate, given the wallet hash is sent as a trait on a device-id profile and not as the id? (src/analytics/index.ts:77-85)
   Options: (A) Deliberate; (B) Alias wallet hash to device id
   Recommendation: Deliberate
142. `chartType`: Which values can chartType take? (src/components/value-chart/Chart.tsx:271)
   Options: (A) line | candlestick; (B) UNCONFIRMED
   Recommendation: Read the ChartType enum in useChartsStore. The values were not in the code shown.
143. `candleResolution`: Which values can candleResolution take? (src/components/value-chart/Chart.tsx:282)
   Options: (A) 1m | 5m | 15m | 1h | 4h | 1d; (B) UNCONFIRMED
   Recommendation: Read the candle resolution type in useChartsStore. The values were not in the code shown.
144. `rnbwMembershipTier`: What are the TierId values? (src/features/rnbw-membership/stores/rnbwMembershipAnalyticsStore.ts:11)
   Options: (A) UNCONFIRMED
   Recommendation: Read the Tier definitions in src/features/rnbw-membership/types.
145. `language`: What strings does language send (the Language enum values)? (src/redux/settings.ts:248)
   Options: (A) en_US | other locales; (B) UNCONFIRMED
   Recommendation: Read the Language enum. Only the default Language.EN_US is shown.
146. `currency`: Which NativeCurrencyKeys values can currency take besides USD? (src/redux/settings.ts:249)
   Options: (A) USD only; (B) USD | EUR | GBP | ...
   Recommendation: Read NativeCurrencyKeys. Only USD is shown.
147. `appIcon`: Which app icon ids exist besides the default 'og'? (src/redux/settings.ts:246)
   Options: (A) og | other icon ids; (B) UNCONFIRMED
   Recommendation: Read the app icon list used by changeIcon.
148. `branchReferringLink`: Can the referring link contain personal data such as emails or ids in its query string? If so, strip it before sending. (src/utils/branch.ts:130)
   Options: (A) No personal data; (B) Strip the query string
   Recommendation: Check the Branch link formats used in campaigns. Strip the query string if unsure.
149. `screenHeight / screenWidth`: Are these meant to hold points or pixels, and are they only meant to describe the first-open device? (src/App.tsx:195-197)
   Options: (A) Points at first open; (B) Refresh on every launch
   Recommendation: Points at first open, as the code does today. A device change won't update them.
150. `Tapped "Cancel" on Wallet Profile modal`: When exactly does `Tapped "Cancel" on Wallet Profile modal` fire? (src/components/expanded-state/WalletProfileState.js:54)
151. `cash_setup_abandoned`: Track Confirm abandoning Cash setup on `Routes.CASH_SETUP_CANCEL_SHEET` with a new event `cash_setup_abandoned`? features already has 5 new events proposed (the budget per PR). (src/features/cash/screens/cash-deposit-setup/components/SetupCancelSheet.tsx#SetupCancelSheet)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: The setup events show intent with no outcome, but not whether the person chose to quit. This fires on confirm only, in the secondary action. It tells you at which step explicit quitters leave.
152. `keychain_check_completed`: Track Run keychain check to restore wallet on `Routes.CHECK_IDENTIFIER_SCREEN` with a new event `keychain_check_completed`? features already has 5 new events proposed (the budget per PR). (src/screens/CheckIdentifierScreen.tsx#CheckIdentifierScreen)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: Answers how many users who run the keychain check have a healthy keychain, and how many hit one of the failure branches. It fires from initAndRunKeychainChecks. It is only logged to the console today.
153. `dapp_browser_session_switched`: Track Switch network for a dapp session on `Routes.DAPP_BROWSER_CONTROL_PANEL` with a new event `dapp_browser_session_switched`? features already has 5 new events proposed (the budget per PR). (src/features/dapp-browser/screens/ControlPanel.tsx#ControlPanel)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: Answers which networks people move a dapp session to, and how many switch while connected. It fires in handleNetworkSwitch before the chainChanged event is injected into the dapp. Wallet switches use the same event with switch_type = wallet.
154. `dapp_browser_site_opened`: Track Browse to a dapp on `Routes.DAPP_BROWSER_SCREEN` with a new event `dapp_browser_site_opened`? features already has 5 new events proposed (the budget per PR). (src/features/dapp-browser/screens/DappBrowser.tsx#DappBrowser)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: The favorite and trending tap events cover tile taps, but a typed URL or search into the browser has no event. This compares how people reach a dapp. Never send the query text, only input_type and the host. The firing point is (UNCONFIRMED), because DappBrowser.tsx has no tracking.
155. `wallet_address_copied`: Track Copy wallet address on `Routes.RECEIVE_MODAL` with a new event `wallet_address_copied`? features already has 5 new events proposed (the budget per PR). (src/screens/ReceiveModal.js#ReceiveModal)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: Answers how many people open Receive and copy the address, rather than only scanning the QR code or sharing it. Other places that copy the address can reuse the event with a different entry_point.
156. `wallet_address_shared`: Track Share wallet address on `Routes.RECEIVE_MODAL` with a new event `wallet_address_shared`? features already has 5 new events proposed (the budget per PR). (src/screens/ReceiveModal.js#ReceiveModal)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: Answers how many people hand out their address from the Receive screen, and how many open the share sheet and back out. ShareButton only calls Share.share and records nothing. Use `entryPoint` so other places that reuse ShareButton can share the event.
157. `rnbw_staking_unstake_warning_acknowledged`: Track Acknowledge exit fee warning and proceed to unstake on `Routes.RNBW_UNSTAKE_SHEET` with a new event `rnbw_staking_unstake_warning_acknowledged`? features already has 5 new events proposed (the budget per PR). (src/features/rnbw-staking/screens/rnbw-unstake-sheet/RnbwUnstakeSheet.tsx#RnbwUnstakeSheet)
   Options: (A) Yes: add it with a written reason; (B) No: leave it untracked
   Recommendation: Answers what share of people who see the exit fee warning continue instead of cancelling. This is the intent step before the existing unstake and unstake-failed outcomes in the code, and without it fee drop-off can't be measured.
158. `discoverPredictionOrderPressed, discoverSectionPressed`: Same `discoverPredictionOrderPressed` and `discoverSectionPressed` calls copy-pasted across layouts: is this real? The verifier couldn't settle it (The copy-paste drift across MarketCarousel, MarketGrid, MarketList and SectionLayout wasn't checked file by file, and the finding asks for no tracking change. It needs an owner's call on refactoring.). (`src/features/discover/components/markets/layouts/MarketCarousel.tsx:87`, `src/features/discover/components/markets/layouts/MarketGrid.tsx:83`, `src/features/discover/components/markets/layouts/MarketList.tsx:82`, `src/features/discover/components/SectionLayout.tsx:41`, `src/features/discover/components/PredictionEventsSection.tsx:87`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it
159. `analyticsTrackingDisabled, analyticsTrackingEnabled`: Analytics toggle events are fired from inside a reducer: is this real? The verifier couldn't settle it (PrivacySection.tsx:30-46 does put device.set, track and enable/disable inside the reducer. Whether React actually re-runs it in production builds (beyond StrictMode in dev) is runtime behaviour.). (`src/screens/SettingsSheet/components/PrivacySection.tsx:30`, `src/screens/SettingsSheet/components/PrivacySection.tsx:35`, `src/screens/SettingsSheet/components/PrivacySection.tsx:42`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it
160. `analytics_tracking.disabled, analytics_tracking.enabled`: analytics_tracking.disabled / enabled are sent from inside a useReducer reducer: is this real? The verifier couldn't settle it (This is the reducer side-effect question of F178. Whether it fires twice depends on runtime.). (`src/screens/SettingsSheet/components/PrivacySection.tsx:35`, `src/screens/SettingsSheet/components/PrivacySection.tsx:42`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it
161. `f2c.provider.flow_completed`: f2c.provider.flow_completed can fire again for the same launch link when the deeplink listener is set up again: is this real? The verifier couldn't settle it (Whether the DeeplinkHandler effect dependencies change while the app is open, and replay the initial URL, is runtime behaviour.). (`src/app/navigation/DeeplinkHandler.tsx:50`, `src/app/navigation/DeeplinkHandler.tsx:54`, `src/app/navigation/DeeplinkHandler.tsx:56`, `src/handlers/deeplinks.ts:172`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it
162. `add_favorite_token`: add_favorite_token: the exchange list calls toggleFavorite from inside a state updater, so it can send twice: is this real? The verifier couldn't settle it (ExchangeAssetList.tsx:229-232 does call toggleFavorite inside the setState updater. A double send needs React to re-run the updater (StrictMode or a replay), which is runtime behaviour.). (`src/components/ExchangeAssetList.tsx:229`, `src/components/ExchangeAssetList.tsx:232`, `src/resources/favorites.ts:177`, `src/resources/favorites.ts:178`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it
163. `claim_claimable.failed`: Sponsored claim 'unresolved status' failure check reads a stale claimStatus: is this real? The verifier couldn't settle it (Whether onSuccess reads a stale claimStatus depends on react-query's options capture at runtime. It can't be shown from the code alone.). (`src/screens/claimables/sponsored/context/SponsoredClaimableContext.tsx:171`, `src/screens/claimables/sponsored/components/SponsoredClaimableFlow.tsx:45`)
   Options: (A) Yes: treat it as a finding and fix it in the migration; (B) No: drop it

## Dropped

- The wallet-sheet tap events have no registry entries of their own: disproved by the verifier (screens.yaml already has entries for all seven events ('Tapped "Done"', '"Cancel" after editing wallet', '"Edit Wallet"', '"Notification Settings"', 'Tapped Delete Wallet', 'final confirm', 'Tapped Edit'), so they are not missing. The wallet_label point is F3.)
- identify() writes chartType and candleResolution to the profile, but the registry has no user properties: disproved by the verifier (user_properties.yaml already registers chartType and candleResolution (operation set, Chart.tsx:271/282), so the registry does have them. That the wrapper merges wallet metadata on identify (index.ts:85) is true but minor.)
- walletAddressHash and walletType are added to every event by analytics.track: disproved by the verifier (walletAddressHash and walletType are listed on every registry entry, not only on Pressed Button. Registering them as super properties is F132.)
- pair_hw_wallet_nav.entered effect returns the result of analytics.track as its cleanup: disproved by the verifier (Arrow-body useEffect returning `analytics.track(...)` is harmless: track returns void (index.ts:102-108), so React gets undefined as the cleanup.)
- Search Query sends the raw text users typed into Discover search: merged, the verifier found it repeats F2: duplicate of F2. This is the same raw-query problem on Search Query.
- Raw token search text is sent as an event property: merged, the verifier found it repeats F2: duplicate of F2.
- Wallet name typed by the user is sent as an event property: merged, the verifier found it repeats F3: duplicate of F3. It is the wallet_label problem at ChangeWalletSheet.tsx:264.
- Wallet name typed by the user is sent as a raw event property: merged, the verifier found it repeats F3: duplicate of F3. The action and buttonName rows on that entry are already marked optional and not sent.
- Raw scanned QR content is sent as an event property: merged, the verifier found it repeats F6: duplicate of F6.
- Wallet group name (raw user text) is sent as an event property: merged, the verifier found it repeats F7: duplicate of F7. Its submit-tap timing point also appears in the registry description.
- Separate cash.*_failed events instead of status + failure_reason on the outcome event (R004): merged, the verifier found it repeats F9: duplicate of F9.
- Raw user-typed ENS names, Unstoppable domains and addresses are sent as event properties: merged, the verifier found it repeats F1: duplicate of F1.
- A 'handled' swap error fires swaps.failed and then swaps.submitted for the same attempt: merged, the verifier found it repeats F4: duplicate of F4.
- Rap completed fires even when the rap failed, and the loop keeps running after a failed action: merged, the verifier found it repeats F22: duplicate of F22.
- Rap completed fires even when the rap failed, so it can't measure completion: merged, the verifier found it repeats F22: duplicate of F22.
- Watched wallet addresses and typed ENS names go out in plain text: merged, the verifier found it repeats F1: duplicate of F1.
- `perpsClosedPosition` can fire and then `perpsClosePositionFailed` fires for the same close: merged, the verifier found it repeats F25: duplicate of F25.
- The raw caught `error` object is sent as an event property: merged, the verifier found it repeats F30: duplicate of F30.
- perps.closed_position fires before the close is confirmed, so one close can also send perps.close_position.failed: merged, the verifier found it repeats F25: duplicate of F25.
- perps.open_position.canceled also counts cancelled adds to an existing position: merged, the verifier found it repeats F26: duplicate of F26.
- Position and asset values are USD amounts but are labelled with the person's native currency: merged, the verifier found it repeats F19: duplicate of F19. Whether totals.total.amount is in USD or the native currency is still unresolved.
- Claim success event can be followed by a failure event for the same claim (rnbw_rewards.claim, rnbw_rewards.claim.failed): merged, the verifier found it repeats F15: duplicate of F15.
- token_launcher.token_created is sent before the launch has finished its follow-up work, so creation_failed can follow it: merged, the verifier found it repeats F13: duplicate of F13.
- swaps.submitted also fires after a 'handled' swap failure: merged, the verifier found it repeats F4: duplicate of F4.
- A 'handled' swap error fires swaps.failed and then swaps.submitted for the same attempt: merged, the verifier found it repeats F4: duplicate of F4.
- Amount properties on swaps.submitted are raw smallest-unit values: merged, the verifier found it repeats F46: duplicate of F46.
- swaps.selected_asset fires on every setAsset call and sends whole asset objects: merged, the verifier found it repeats F44: duplicate of F44.
- rnbw_rewards.claim cannot tell a wallet claim from a stake-rewards claim: merged, the verifier found it repeats F28: duplicate of F28.
- Airdrop claim failures include user cancellations and send the raw exception text: merged, the verifier found it repeats F37: duplicate of F37.
- Buy-order completion is silently not tracked when any amount or timestamp field is missing: merged, the verifier found it repeats F47: duplicate of F47.
- wallet_create.failed sends the raw exception message as its failure reason: merged, the verifier found it repeats F14: duplicate of F14.
- cash.buy_completed is silently skipped for a completed order missing a field, while the UI shows success: merged, the verifier found it repeats F47: duplicate of F47.
- Trending dapp click is never sent for the first tile, recents or the featured card: merged, the verifier found it repeats F53: duplicate of F53. Recents cards (Homepage.tsx:289) pass no index, so they never send it either.
- rnbw_airdrop.claim is skipped, and claim.failed sent, when the post-claim balance refresh throws: merged, the verifier found it repeats F15: duplicate of F15.
- add_wallet_flow.started: cloud restore is reported as type 'seed', hardware wallet hard-codes isFirstWallet false: merged, the verifier found it repeats F55: duplicate of F55.
- learn_article.opened is sent when the screen closes, not when the article opens: merged, the verifier found it repeats F50: duplicate of F50.
- claim_claimable.failed counts user-cancelled transactions as failures: merged, the verifier found it repeats F37: duplicate of F37.
- token_list from Discover counts tokens that HAVE a price as no_price: merged, the verifier found it repeats F51: duplicate of F51.
- All 29 `direct_sdk` findings are a script artefact: `@/analytics` is the wrapper, not the SDK: merged, the verifier found it repeats F69: duplicate of F69.
- direct_sdk findings point at the project's own tracking wrapper (script artefact): merged, the verifier found it repeats F69: duplicate of F69.
- direct_sdk findings are a script artefact: `analytics` from '@/analytics' is the wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- Event and property names break the naming pattern (R001, R007): merged, the verifier found it repeats F70: duplicate of F70.
- direct_sdk flags are a script artefact: `analytics.track` is the tracking wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- Property names are mixed camelCase and snake_case, and some types differ between events: merged, the verifier found it repeats F70: duplicate of F70. The R013 type conflicts remain unchecked.
- Naming-pattern violations (R001/R007) come from the app's established dotted, camelCase convention: merged, the verifier found it repeats F70: duplicate of F70.
- Legacy event and property names break the naming pattern: merged, the verifier found it repeats F70: duplicate of F70.
- Dotted event names and camelCase property names break the plan's snake_case patterns: merged, the verifier found it repeats F70: duplicate of F70.
- camelCase property names come from shared analytics code, not from these events (R007): merged, the verifier found it repeats F74: duplicate of F74.
- direct_sdk findings are false positives: these calls go through the wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- camelCase property names (`componentStack`, `eventId`, `walletAddressHash`, `walletType`) fail the property naming pattern: merged, the verifier found it repeats F74: duplicate of F74.
- Mixed event and property naming (R001/R007), a legacy-vs-new split in the plan: merged, the verifier found it repeats F70: duplicate of F70.
- direct_sdk findings are a script artefact: `analytics` is the tracking wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- Naming hygiene: mixed casing and per-screen events: merged, the verifier found it repeats F70: duplicate of F70.
- walletAddressHash, walletType and durationInMs are camelCase; the wallet two are added by the analytics wrapper, not by these call sites: merged, the verifier found it repeats F74: duplicate of F74.
- Naming rule failures come from the repo's own convention, not from broken events: merged, the verifier found it repeats F70: duplicate of F70.
- All 30 property_missing findings are script artefacts: the scanner attached the neighbouring event's call to each registry entry: merged, the verifier found it repeats F72: duplicate of F72. The real calls I read (claimAirdrop.ts:49/91/116, PolymarketNewPositionSheet:209) match.
- Naming-pattern failures are inherited names, not a tracking bug: merged, the verifier found it repeats F70: duplicate of F70.
- property_missing findings are script artefacts: the real calls send entryMode, reason and mode: merged, the verifier found it repeats F72: duplicate of F72.
- All 30 property_missing flags are scanner artefacts: the checks read the wrong tracking call in each file: merged, the verifier found it repeats F72: duplicate of F72. ClosePositionBottomSheet.tsx:150 and :179 send the properties claimed.
- Wallet profile and app-icon events carry little to break down by: merged, the verifier found it repeats F40: duplicate of F40. The CTA delay part is F100.
- property_missing flags are script artefacts: the flagged lines are not these events' calls: merged, the verifier found it repeats F72: duplicate of F72.
- Failure is a separate event, and the stage of the failure can't be told apart: merged, the verifier found it repeats F8: duplicate of F8. The missing `stage` property is an addition.
- walletAddressHash and walletType are camelCase default properties added to every event: merged, the verifier found it repeats F74: duplicate of F74.
- All 30 property_missing flags are script artefacts: the scanner read the wrong call: merged, the verifier found it repeats F72: duplicate of F72. AddWalletSheet.tsx:45/49 and ChangeWalletSheet.tsx:264/267 match.
- Event names use dots, spaces and quotes: legacy names that the pattern flags (R001): merged, the verifier found it repeats F70: duplicate of F70.
- Naming-pattern errors on app_store_review.prompted come from the repo's own convention, not a tracking bug: merged, the verifier found it repeats F70: duplicate of F70.
- Registry entries list properties that belong to other events: merged, the verifier found it repeats F75: duplicate of F75.
- Missing-property findings are script artefacts: the scanner linked each event to a neighbouring call: merged, the verifier found it repeats F72: duplicate of F72. actions.ts:689 sends category, failed_action and label, and SignTransactionSheet:295 is the reject call.
- status without failure_reason on the predictions and rnbw claim events (R012): merged, the verifier found it repeats F8: duplicate of F8.
- All 28 property_missing flags are script artefacts: the scanner matched other events' calls to these three events: merged, the verifier found it repeats F72: duplicate of F72. useImportingWallet.ts:170/200/222 send exactly address+input, address+input, ens+input.
- Registry drafts list properties the code never sends, and omit that `address` is not sent on the read-only event: merged, the verifier found it repeats F75: duplicate of F75.
- Script artefact: the property_missing errors come from unrelated calls in the same file: merged, the verifier found it repeats F72: duplicate of F72. useWalletCloudBackup.ts:131-138 and :152 send what the registry lists.
- property_missing errors are scanner artefacts: the scripts matched neighbouring tracking calls, not the real ones: merged, the verifier found it repeats F72: duplicate of F72.
- Event and property names break the snake_case `object_action` pattern (R001/R007): merged, the verifier found it repeats F70: duplicate of F70.
- Two events fire for one tap on the add-wallet sheet: merged, the verifier found it repeats F55: duplicate of F55.
- camelCase properties fail the property naming rule; two of them are added by the analytics wrapper, not by this call: merged, the verifier found it repeats F74: duplicate of F74.
- Three Discover events use legacy human-readable names instead of the object_action pattern: merged, the verifier found it repeats F70: duplicate of F70.
- R007 on add_favorite_token is a naming-convention mismatch: the properties are camelCase like the rest of the codebase, and two are added by the analytics client: merged, the verifier found it repeats F74: duplicate of F74.
- Event names debug.identify_probe and 'First App Open' don't match the naming pattern: merged, the verifier found it repeats F70: duplicate of F70. The probe-removal part is F94.
- Script artefact: token_list and swaps.failed property gaps were checked against the wrong calls: merged, the verifier found it repeats F72: duplicate of F72.
- property_missing is a script artefact: the swaps.failed/swaps.submitted properties are built in a wrapper, and the scanner pointed at the wrong call: merged, the verifier found it repeats F72: duplicate of F72. trackSwapEvent.ts:47-71 builds the full set, and line 1023 is selected_asset.
- property_missing findings on swaps.submitted and swaps.selected_asset are scanner artefacts: merged, the verifier found it repeats F72: duplicate of F72.
- property_missing flags are registry-draft errors, not missing tracking in the code: merged, the verifier found it repeats F75: duplicate of F75.
- Phantom cross-properties on the two chart events (script artefact): merged, the verifier found it repeats F75: duplicate of F75.
- property_missing flags on cash.kyc_awaiting_decision and cash.kyc_failed are script artefacts: every call sends its required property: merged, the verifier found it repeats F72: duplicate of F72.
- cash.card_linked / cash.card_link_failed: the two calls' references are swapped, so each looks like it lacks the other's property: merged, the verifier found it repeats F75: duplicate of F75.
- property_missing on cash.buy_completed / cash.buy_failed is a script artefact: the checks compared the wrong call sites: merged, the verifier found it repeats F72: duplicate of F72. cashBuyOrderStore.ts:123 and :146 send their own properties.
- property_missing flags on Rap failed / Rap started are script artefacts (wrong call attribution): merged, the verifier found it repeats F72: duplicate of F72. actions.ts:689 and :717 match.
- Rap started/completed 'label' flags are a script artefact: the real calls send label: merged, the verifier found it repeats F72: duplicate of F72. actions.ts:717 and :742 send label, and 394-446 are profile events.
- WalletConnect property_missing findings are attribution errors: the scanner assigned calls to the wrong event: merged, the verifier found it repeats F72: duplicate of F72.
- property_missing flags are a script artefact: they point at other events' track calls: merged, the verifier found it repeats F72: duplicate of F72.
- Registry drafts for the backup error events carry stale or unconfirmed property details: merged, the verifier found it repeats F75: duplicate of F75. useWalletCloudBackup.ts:152 sends no `error`.
- Asset-update events: the registry has properties the calls never send, so the checker reports them missing: merged, the verifier found it repeats F75: duplicate of F75.
- property_missing is a scanner artefact: the two calls were cross-attributed to both events: merged, the verifier found it repeats F75: duplicate of F75.
- Event names are Title Case with spaces, so they fail R001: merged, the verifier found it repeats F70: duplicate of F70.
- property_missing findings are a script and registry artefact: the two signing events were swapped and cross-listed: merged, the verifier found it repeats F75: duplicate of F75. performance.ts:62 sends `log` and :69-75 sends the end-of-flow fields.
- Dotted event names (`error_boundary.viewed`, `error_boundary.reset`) don't match the plan's naming pattern: merged, the verifier found it repeats F70: duplicate of F70.
- Wallet create events use a dotted name the naming pattern rejects (script/convention mismatch, not a wallet bug): merged, the verifier found it repeats F70: duplicate of F70.
- Wallet Profile submit and cancel have no start event, so no rate can be computed: merged, the verifier found it repeats F40: duplicate of F40.
- Send amount-input events fire on every keystroke and carry no value information: merged, the verifier found it repeats F34: duplicate of F34.
- probe/value 'missing' is a script artefact: the flagged line is the First App Open call, not the probe calls: merged, the verifier found it repeats F72: duplicate of F72.
- swaps.failed errorMessage is free text from the swap error: merged, the verifier found it repeats F87: duplicate of F87.
- Script artefact: the property check read the wrong call site (swaps.selected_asset), not swaps.submitted: merged, the verifier found it repeats F72: duplicate of F72.
- Registry entry for swaps.submitted lists properties it never sends (asset, otherAsset, parameters): merged, the verifier found it repeats F75: duplicate of F75.
- Property-missing errors on Pressed Button are scanner misattribution: merged, the verifier found it repeats F72: duplicate of F72. Pressed Button still carries a stray wallet_label row.
- Scanner attributed other events' calls to cash.phone_verify_failed; the two real calls are complete: merged, the verifier found it repeats F72: duplicate of F72.
- cash.kyc_failed 'reason missing' is a script artefact: the flagged lines are other KYC events: merged, the verifier found it repeats F72: duplicate of F72.
- cash.sign_in_failed 'reason missing' is a script artefact: the flagged lines are the submitted, succeeded and cancelled calls: merged, the verifier found it repeats F72: duplicate of F72. cashSignInService.ts:53/68/76 send only trigger, and :78-81 sends trigger and reason.
- cash.buy_completed 'properties missing' is a script artefact: the flagged lines are cash.buy_failed calls: merged, the verifier found it repeats F72: duplicate of F72.
- cash.phone_resend_failed 'reason' flagged missing at a call that belongs to cash.phone_submitted: merged, the verifier found it repeats F72: duplicate of F72.
- perps.opened_position property_missing flags are a script artefact: they point at the canceled event's call: merged, the verifier found it repeats F72: duplicate of F72.
- rnbw_rewards.claim "missing properties" come from the failure call, not from a bug: merged, the verifier found it repeats F72: duplicate of F72. claimRewards.ts:123 is the success call and :151 the failed call.
- property_missing flags are a script artefact: only line 222 sends this event: merged, the verifier found it repeats F72: duplicate of F72.
- property_missing flags on 'Show wallet profile modal for imported wallet' are a script artefact: they point at other events' calls: merged, the verifier found it repeats F72: duplicate of F72. Line 279 sends address and type. The neighbouring events already have registry entries.
- property_missing flags on 'Scanned broken or unsupported QR code' are a script artefact: they point at other scan events: merged, the verifier found it repeats F72: duplicate of F72.
- Derived wallet address is sent raw while the wrapper already adds a hash: merged, the verifier found it repeats F1: duplicate of F1. Its evidence, line 279, is already in F1.
- property_missing on backup 'label' is a script artefact: the flagged calls are different events: merged, the verifier found it repeats F72: duplicate of F72. 'View how to Enable iCloud' and 'Ignore how to enable iCloud' already have entries in hooks.yaml.
- add_wallet_flow.started mixes methods and hard-codes isFirstWallet: merged, the verifier found it repeats F55: duplicate of F55.
- direct_sdk findings are a script artefact: `analytics` is the project's own wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- NFT detail event sends the NFT name and image URL and fires for every NFT viewed for 5 seconds: merged, the verifier found it repeats F60: duplicate of F60.
- direct_sdk flags are a script artefact: these calls already go through the project's tracking wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- direct_sdk findings are a script artefact: these calls already go through the tracking wrapper: merged, the verifier found it repeats F69: duplicate of F69.
- tokenList event re-fires whenever the asset list changes: merged, the verifier found it repeats F45: duplicate of F45.
- f2cProviderFlowCompleted treats a deeplink return as a success for Ramp, and has no outcome for other providers: merged, the verifier found it repeats F61: duplicate of F61.
- Temporary debug probe event is still live: merged, the verifier found it repeats F94: duplicate of F94.
- Event name uses a dotted namespace, not the object_action snake_case pattern (R001): merged, the verifier found it repeats F70: duplicate of F70.
- Registry draft for wallet_create.succeeded lists an `error` property the code never sends: merged, the verifier found it repeats F75: duplicate of F75. wallet.ts:722 sends analyticsParams with no error key.
- Search Query fires only when the input is cleared, not once per search: merged, the verifier found it repeats F52: duplicate of F52.
- add_new_wallet_group.name fires on the submit tap, even with an empty name, and sends the typed name: merged, the verifier found it repeats F7: duplicate of F7.
- token_details.nft fires on unmount as well as at 5 seconds, and the cancelled flag is ignored: merged, the verifier found it repeats F60: duplicate of F60.
- request.rejected mixes user rejections with signing failures, and skips some failures: merged, the verifier found it repeats F32: duplicate of F32.
- Changed Global Notification Settings is unreachable today: no global topics exist: merged, the verifier found it repeats F49: duplicate of F49.
- Received wc connection sends waitingTime as a number or an error string: merged, the verifier found it repeats F21: duplicate of F21.
- token_launcher.image_upload_failed sends url and isModerated that are always undefined: merged, the verifier found it repeats F57: duplicate of F57.
- token_launcher.share_pressed fires after the share sheet closes, whether or not the person shared: merged, the verifier found it repeats F97: duplicate of F97.
- token_list fires several times per visit: merged, the verifier found it repeats F54: duplicate of F54.
- watch_wallet sends the raw wallet input as `address` in the derived-account branch: no evidence that can be re-checked (src/hooks/useImportingWallet.ts:270-275  if (type === 'watch') { analytics.track(analytics.event.watchWallet, { addressOrEnsName: ens, address: input }); } doesn't exist; src/hooks/useImportingWallet.ts:277  startImportProfile(name || '', forceColor, walletResult.address, finalAvatarUrl); doesn't exist)
- Show wallet profile modal for read only wallet is skipped whenever the ENS reverse lookup fails, though the flow continues: no evidence that can be re-checked (src/hooks/useImportingWallet.ts:213-228  ens = await fetchReverseRecord(input); ... analytics.track(analytics.event.showWalletProfileModalForReadOnlyWallet, { ens, input }); } catch (e) { logger.error(...) } doesn't exist; src/hooks/useImportingWallet.ts:239  startImportProfile(name || '', forceColor, input, finalAvatarUrl); doesn't exist)
- mined_transaction.assets_timed_out can be sent again on the next poll for the same transaction: no evidence that can be re-checked (src/hooks/useWatchAssetUpdates.ts:69-73 (analytics.track(event.minedTransactionAssetsTimedOut ...) inside the filter callback) doesn't exist; src/hooks/useWatchAssetUpdates.ts:129 (await getPlatformClient().get('/assets/GetAssetUpdates')) doesn't exist; src/hooks/useWatchAssetUpdates.ts:161 (removeWatchedTransactions runs only after the request succeeds) doesn't exist; src/hooks/useWatchAssetUpdates.ts:183-184 (catch only logs) doesn't exist)

<sub>Datalyze Tracking Agent · method 2026.10.19 · claude-sonnet-5-5 (130 calls), claude-opus-5-5 (1 call) (4 failed, counted) · 12,454,538 tokens in, 411,093 out · $14.20<br>Cost cap for the install audit: $35.00 (`install_audit.max_cost_usd` in .datalyze/config.yaml).</sub>
