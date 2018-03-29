# Proposed Redux State

## Table of Contents
1. [Helper Types](#Helper Types)
2. [Root State](#Root State)
3. [Edge](#Edge)
4. [Account](#Account)
5. [Password Reminder](#Password Reminder)
6. [Wallet Settings](#Wallet Settings)
7. [Currency Settings](#Currency Settings)
8. [Local Settings](#Local Settings)
9. [Synced Settings](#Synced Settings)
10. [Settings](#Settings)
11. [Wallets](#Wallets)
12. [Scenes](#Scenes)
13. [Device](#Device)
14. [Exchange](#Exchange)
15. [Send](#Send)
16. [Request](#Request)

### Helper Types
```javascript
type WalletId = string
type CurrencyCode = string
type IsoCurrencyCode = string
type NativeAmount = string
type DisplayAmount = string
type ExchangeAmount = string
type FiatAmount = string
type GuiAmount = {
  native: NativeAmount,
  display: DisplayAmount,
  exchange: ExchangeAmount,
  fiat: (CurrencyConverter, IsoCurrencyCode) => FiatAmount // ??? keto-derived?
}
type FeesSettings = Object
type PluginName = string
type SceneKey = string
type SceneState = Object
```

### Root State
```javascript
export type State = {
  edge: EdgeState, // everything non-serializable
  account: AccountState,
  wallets: WalletsState,
  settings: SettingsState,
  scenes: ScenesState,
  device: DeviceState,
  exchange: ExchangeState,
  send: SendState,
  request: RequestState
}
```

### Edge
```javascript
export type EdgeState = { // everything non-serializable
  account: EdgeAccount | null,
  context: EdgeContext | null,
  wallets: {[WalletId]: EdgeCurrencyWallet} // keto-derived from account?
}
```

### Account
```javascript
export type LoginTypeState = ‘newAccount’
| ‘password’
| ‘pin’
| ‘recovery’
| ‘key’,
| ‘edge’,

export type AccountState = {
  username: string, // keto-derived from state.edge.account.username
  loginType: LoginTypeState,pass // keto-derived from state.edge.account
  passwordReminder: PasswordReminderState
}
```

### Password Reminder
```javascript
export type PasswordReminderState = {
  needsPasswordCheck: boolean,
  lastPasswordUse: Date,
  nonPasswordDaysRemaining: number,
  nonPasswordLoginsRemaining: number,
  nonPasswordDaysLimit: number,
  nonPasswordLoginsLimit: number
}
```

### Wallet Settings
```javascript
export type WalletSettingsState = {
  [WalletId]: {
    privacyMode: { // used to show or hide wallet balance in fiat
      isEnabled: boolean
    }
  }
}
```

### Currency Settings
```javascript
export type CurrencySettingsState = {
  [CurrencyCode]: {
    denominations: {[key: string]: EdgeDenomination},
    displayDenominationKey: string, // overkill? confusing?
    displayDenomination: EdgeDenomination, // keto-derived
    exchangeDenomination: EdgeDenomination, // keto-derived
    spendingLimits: {
      daily: {
        isEnabled: boolean,
        nativeAmount: string
      },
      transaction: {
        isEnabled: boolean,
        nativeAmount: string
      }
    }
  }
}
```

### Local Settings
```javascript
export type LocalSettingsState = {
  bluetoothMode: {
    isEnabled: boolean,
    isSupported: boolean,
  },
  otpMode: {
    isEnabled: boolean,
    key: string,
    resetDate: Date,
    daysRemaining: Date
  },
  touchId: {
    isEnabled: boolean,
    isSupported: boolean
  }
}
```

### Synced Settings
```javascript
export type SyncedSettingsState = {
  autoLogoutMode: {
    isEnabled: boolean,
    seconds: number
  },
  defaultFiat: IsoCurrencyCode,
  merchantMode: {
    isEnabled: boolean
  },
  privacyMode: { // used to show or hide total account balance in fiat
    isEnabled: boolean,
  },
  byWalletId: WalletSettingsState,
  byCurrencyCode: CurrencySettingsState
}
```

### Settings
```javascript
export type SettingsState = {
  autoLogoutMode: SyncedSettingsState.autoLogoutMode, // keto-derived
  bluetoothMode: LocalSettings.bluetoothMode, // keto-derived
  defaultFiat: SyncedSettingsState.defaulFiat, // keto-derived
  merchantMode: SyncedSettingsState.merchantMode, // keto-derived
  otpMode: LocalSettings.otpMode // keto-derived
  privacyMode: SyncedSettings.privacyMode // keto-derived
  touchId: { LocalSettings.touchId // keto-derived
  byWalletId: SynchedSettingsState.byWalletId, // keto-derived
  byCurrencyCode: SyncedSettings.byCurrencyCode, // keto-derived
  localSettings: LocalSettingsState, // store in EdgeState?
  syncedSettings: SyncedSettingsState // store in EdgeState?
}
```

### Wallets
```javascript
export type WalletsState = {
  byId: {[WalletId]: GuiWallet},
  activeWalletIds: Array<WalletId>,
  archivedWalletIds: Array<WalletId>,
  selectedWalletId: WalletId
  selectedCurrencyCode: CurrencyCode
  selectedWallet: GuiWallet // derived from byId[selectedWalletId]
}
```

### Scenes
```javascript
export type ScenesState = { // INCOMPLETE, JUST AN EXAMPLE
  [SceneKey]: SceneState,
  main: {
    dropdownAlert: { // displays globally
      isVisible: boolean
    },
    popupModal: { // displays globally
      isVisible: boolean
    },
    passwordReminderModal: { // displays globally
      isVisible: boolean
    },
  },
  walletList: {
    deleteWalletModal: { // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    privateSeedModal: { // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    renameWalletModal: { // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
    resyncWalletModal: { // displays only on this scene
      isVisible: boolean,
      walletId: WalletId
    },
  }
}
```

### Device
```javascript
export type DeviceState = {
  contacts: ContactsState,
  locale: LocaleState,
  permissions: PermissionsState,
  specs: SpecsState
}

export type PermissionState = ‘granted’
  | ‘denied’
  | ‘restricted’
  | null
  export type Permission = ‘bluetooth’
  | ‘camera’
  | ‘contacts’
  | ‘photos’
  export type PermissionsState = {
  [Permission]: PermissionState
}

export type ContactsState = Array<GuiContact> | null
export type LocaleState = {
  localeIdentifier: string,
  decimalSeparator: string,
  quotationBeginDelimiterKey: string,
  quotationEndDelimiterKey: string,
  currencySymbol: string,
  currencyCode: string,

  // ios only:
  usesMetricSystem: boolean,
  localeLanguageCode: string,
  countryCode: string,
  calendar: string,
  groupingSeparator: string,
  collatorIdentifier: string,
  alternateQuotationBeginDelimiterKey: string,
  alternateQuotationEndDelimiterKey: string,
  measurementSystem: string,
  preferredLanguages: Array<string>
} | null

export type SpecsState = {
  // android only
  // apiLevel: number,
  // firstInstallTime: number,
  // ipAddress: Promise<string>,
  // instanceId: string,
  // lastUpdateTime: number,
  // macAddress: Promise<string>,
  // maxMemory: number,
  // phoneNumber: string,
  // serialNumber: string,
    applicationName: string,
  brand: string,
  buildNumber: string,
  bundleId: string,
  carrier: string,
  deviceCountry: string,
  deviceId: string,
  deviceLocale: string,
  deviceName: string,
  freeDiskStorage: number,
  manufacturer: string,
  model: string,
  readableVersion: string,
  systemName: string,
  systemVersion: string,
  timezone: string,
  totalDiskCapacity: number,
  totalMemory: number,
  uniqueId: string,
  userAgent: string,
  version: string,
  is24Hour: boolean,
  isEmulator: string,
  isPinOrFingerprintSet: boolean,
  isTablet: boolean
} | null
```

### Exchange
```javascript
export type ExchangeInfo = {
  walletId: Id,
  currencyCode: CurrencyCode,
  amount: GuiAmount
}

export type ExchangeState = {
  source: ExchangeInfo | null,
  destination: ExchangeInfo | null,
  transaction: EdgeTransaction | null
  error: Error | null
}
```

### Send
```javascript
export type SendInfo = {
  walletId: Id, // possibly keto-derived
  currencyCode: CurrencyCode, // possibly keto-derived
  amount: GuiAmount
}

export type SendState = {
  uri: EdgeParsedUri | null,
  spendInfo: EdgeSpendInfo | null, // keto-derived
  source: SendInfo,
  destination: {
    address: string
  },
  feeSettings: FeesSettings | null,
  transaction: EdgeTransaction | null,
  metadata: EdgeMetadata | null,
  error: Error | null
}
```

### Request
```javascript
export type RequestInfo = {
  walletId: Id, // possibly keto-derived
  currencyCode: CurrencyCode, // possibly keto-derived
  amount: GuiAmount
}

export type RequestState = {
  destination: RequestInfo,
  amountCurrent: GuiAmount,
  amountRemaining: GuiAmount // keto-derived
}
```
