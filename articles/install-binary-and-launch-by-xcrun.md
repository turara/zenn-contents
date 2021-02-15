---
title: '【iOS】コマンドラインからバイナリをインストールする。ReactNativeの再ビルド（が不要なとき）はすっ飛ばそう'
emoji: '📲'
type: 'tech' # tech: 技術記事 / idea: アイデア
topics: [iOS, ReactNative]
published: true
---

# Conclusion

```shell
# インストール
xcrun simctl install your-device-udid your-binary-path
# 起動
xcrun simctl launch your-device-udid your-app-bundleid
```

※ こちらのコマンド自体は ReactNative とは関係ありません。

# Why?

ReactNative の開発で iOS 端末やシミュレータを使っていると、ときどき困ることがあります。

`react-native run-ios`でアプリを実行してしばらくは調子良く動いているのに、ふとした拍子にリロード（Cmd+R）やシェイク（Cmd+Ctrl+Z）が効かなくなって再度`run-ios`を実行するハメになり、無駄に長いビルド待ちが発生してしまう……。

再ビルドの必要がないときはバイナリだけ再インストールしちゃいましょう！

# How?

`react-native run-ios`が呼びだしているインストールと起動のコマンドを直接呼んじゃいます。

https://github.com/react-native-community/cli/blob/56a74c0d0b0cbd50816c0f4027e59c47e5181be0/packages/platform-ios/src/commands/runIOS/index.ts#L207

```typescript:runIOS/index.ts
logger.info(`Installing "${chalk.bold(appPath)}"`);

child_process.spawnSync(
  'xcrun',
  ['simctl', 'install', selectedSimulator.udid, appPath],
  {stdio: 'inherit'},
);

...

logger.info(`Launching "${chalk.bold(bundleID)}"`);

const result = child_process.spawnSync('xcrun', [
  'simctl',
  'launch',
  selectedSimulator.udid,
  bundleID,
]);
```

この部分を読むと、インストールには端末の UDID とバイナリのパス、起動には端末の UDID とアプリの BundleID が必要なことがわかります。

## バイナリのパスを知る

`react-native run-ios`を実行した場合には最後の方にパスが出力されています。

```shell
info Installing build/YourProjectName/Build/Products/Debug-iphonesimulator/YourAppName.app
```

## 端末の UDID を知る

下記のコマンドで端末とその UDID の一覧が表示されます

```shell
xcrun xctrace list devices
```

## インストールとアプリ起動

プロジェクトのルートから実行する場合は先程のバイナリのパスに`ios`をつけて……

```shell
# インストール
xcrun simctl install your-device-udid ios/build/YourProjectName/Build/Products/Debug-iphonesimulator/YourAppName.app
# 起動
xcrun simctl launch your-device-udid your-app-bundleid
```

# 環境

- Xcode 12.4
