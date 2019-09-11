
# React Native Virgil Crypto

React Native bridge for Virgil Security Crypto Library for [iOS](https://github.com/VirgilSecurity/virgil-crypto-x) and [Android](https://github.com/VirgilSecurity/virgil-sdk-java-android).

## Library purposes

* Be a replacement for [virgil-crypto-javascript](https://github.com/VirgilSecurity/virgil-crypto-javascript) in React Native projects
* Asymmetric Key Generation
* Encryption/Decryption of data
* Generation/Verification of digital signatures

## Usage

This library's API is compatible with the [virgil-crypto for JavaScript](https://github.com/VirgilSecurity/virgil-crypto-javascript) and can be used in place of the latter in React Native projects. The main difference is that in JS library a class named `VirgilCrypto` is exported from the module that you need to create instances of, whereas this library exports an "instance" of that class ready to be used. Also, stream encryption is not available as there is no stream implementation in React Native. We're investigating the options to support file encryption though.

### Generate a key pair

Generate a Private Key with the default algorithm (ED25519):

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const keyPair = virgilCrypto.generateKeys();
```

### Generate and verify a signature

Generate signature and sign data with a private key:

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const signingKeypair = virgilCrypto.generateKeys();

// prepare a message
const messageToSign = 'Hello, Bob!';

// generate a signature
const signature = virgilCrypto.calculateSignature(messageToSign, signingKeypair.privateKey);
// signature is a NodeJS Buffer polyfill
console.log(signature.toString('base64'));
```

Verify a signature with a public key:

```javascript
// verify a signature
const isVerified = virgilCrypto.verifySignature(messageToSign, signature, signingKeypair.publicKey);
```

### Encrypt and decrypt data

Encrypt Data on a Public Key:

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const encryptionKeypair = virgilCrypto.generateKeys();

// prepare a message
const messageToEncrypt = 'Hello, Bob!';

// encrypt the message
const encryptedData = virgilCrypto.encrypt(messageToEncrypt, encryptionKeypair.publicKey);
// encryptedData is a NodeJS Buffer polyfill
console.log(encryptedData.toString('base64'));
```

Decrypt the encrypted data with a Private Key:

```javascript
// decrypt the encrypted data using a private key
const decryptedData = virgilCrypto.decrypt(encryptedData, encryptionKeypair.privateKey);

// convert Buffer to string
const decryptedMessage = decryptedData.toString('utf8');
```

### File encryption

To encrypt a file you will need to know its location in the file system. For images you can use [React Native API](https://facebook.github.io/react-native/docs/cameraroll.html), or a library such as [react-native-image-picker](https://github.com/react-native-community/react-native-image-picker) or [react-native-camera-roll-picker](https://github.com/jeanpan/react-native-camera-roll-picker).

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const keypair = virgilCrypto.generateKeys();

// this must be defined in your code
pickAnImage()
.then(image => {
  return virgilCrypto.encryptFile({
    // assuming `image` has a `uri` property that points to its location in file system
    inputPath: image.uri, 
    // This can be a custom path that your application can write to
    // e.g. RNFetchBlob.fs.dirs.DocumentDir + '/encrypted_uploads/' + image.fileName,
    // If not specified, a temporary file will be created.
    outputPath: undefined,
    publicKeys: keypair.publicKey
  })
  .then(encryptedFilePath => {
    // encryptedFilePath is the location of the encrypted file in the file system
    // the original image file remain intact
    // you can now upload this file using `fetch` and `FormData`, e.g.:
    const data = new FormData();
    data.append('photo', {
      uri: 'file://' + encryptedFilePath,
      type: image.type,
      name: image.fileName
    });

    return fetch(url, { method: 'POST', body: data });
  });
});
```

Decryption works similarly to encryption - you provide a path to the encrypted file in the file system (network urls are not supported). You can use a library such as [rn-fetch-blob](https://github.com/joltup/rn-fetch-blob) or [react-native-fs](https://github.com/itinance/react-native-fs) to download a file directly into file system.

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const keypair = virgilCrypto.generateKeys();

// this must be defined in your code
downloadImage()
.then(downloadedFilePath => {
  return virgilCrypto.decryptFile({
    inputPath: downloadedFilePath, 
    // This can be a custom path that your application can write to
    // e.g. RNFetchBlob.fs.dirs.DocumentDir + '/decrypted_downloads/' + image.id + '.jpg';
    // If not specified, a temporary file will be created
    outputPath: undefined,
    privateKey: keypair.privateKey
  })
  .then(decryptedFilePath => {
    return <Image src={`file://${decryptedFilePath}`} />
  });
});
```

It is also possible to calculate the digital signature of a file

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const keypair = virgilCrypto.generateKeys();
// this must be defined in your code
pickAnImage()
.then(image => {
  return virgilCrypto.generateFileSignature({
    inputPath: image.uri, 
    privateKey: keypair.privateKey
  })
  .then(signature => ({ ...image, signature: signature.toString('base64') }));
});
```

And verify the signature of a file

```javascript
import { virgilCrypto } from 'react-native-virgil-crypto';

const keypair = virgilCrypto.generateKeys();

// this must be defined in your code
downloadImage()
.then(downloadedFilePath => {
  return virgilCrypto.verifyFileSignature({
    inputPath: image.downloadedFilePath, 
    signature: image.signature, 
    publicKey: keypair.publicKey
  })
  .then(isSigantureVerified => ({ ...image, isSigantureVerified }));
});
```

See the [sample project](https://github.com/VirgilSecurity/react-native-virgil-crypto/tree/master/examples/FileEncryptionSample) for a complete example of working with encrypted files.

### Working with binary data

All of the methods of `virgilCrypto` object that accept binary data, accept them in the form of `string` or `Buffer`. All of the methods that return binary data, return them in the form of `Buffer`. We use [this library](https://github.com/feross/buffer) as the native implementation is not available in react native. We re-export the `Buffer` from the module for your convenience:

```javascript
import { Buffer } from 'react-native-virgil-crypto';

console.log(Buffer.from('hello Buffer').toString('base64')); // prints aGVsbG8gQnVmZmVy
```

### Performance

See the [sample project](https://github.com/VirgilSecurity/react-native-virgil-crypto/tree/master/examples/Benchmarks) for a complete example that you can use to measure performance of this library on your own devices.

### Usage with `virgil-sdk`

If you want this package to work with `virgil-sdk`, you'll need:
  1. Update the `virgil-sdk` to version 6.0.0, which is currently a pre-release. You can install it with `npm install virgil-sdk@next`.
  2. Install one more package - `npm install @virgilsecurity/sdk-crypto`
  3. Update the `@virgilsecurity/key-storage-rn` to the latest version, which is also currently in pre-release - `npm install @virgilsecurity/key-storage-rn@next`
  4. Install the `react-native-keychain` package, a dependency of `@virgilsecurity/key-storage-rn`

Here's an example of initialization:

```js
import { virgilCrypto } from 'react-native-virgil-crypto';
import createNativeKeyEntryStorage from '@virgilsecurity/key-storage-rn/native';
import { VirgilCardCrypto, VirgilPrivateKeyExporter } from '@virgilsecurity/sdk-crypto';
import { CachingJwtProvider, CardManager, VirgilCardVerifier, PrivateKeyStorage } from 'virgil-sdk';

const jwtProvider = new CachingJwtProvider(getTokenFunctionDefinedInYourCode);
const cardCrypto = new VirgilCardCrypto(virgilCrypto);
const cardVerifier = new VirgilCardVerifier(cardCrypto);
const cardManager = new CardManager({
  cardCrypto,
  cardVerifier,
  accessTokenProvider: jwtProvider,
  retryOnUnauthorized: true,
});

const privateKeyStroage = new PrivateKeyStorage(
  new VirgilPrivateKeyExporter(virgilCrypto),
  createNativeKeyEntryStorage({ username: nameOfTheStorageChosenByYou });
);

// use `cardManager` to work with cards
// use `privateKeyStorage` to save and retrieve private keys to\from local storage
```

## Getting started

`$ npm install react-native-virgil-crypto --save`

### iOS Setup

This library depends on the native [virgil-crypto-x](https://github.com/VirgilSecurity/virgil-crypto-x) project. To keep the React Native library agnostic of your dependency management mehtod, the native libraries are not distributed as part of the bridge.

VirgilCrypto supports two options for dependency management:

1. **CocoaPods**

    **RN<0.60**

    ```sh
    react-native link react-native-virgil-crypto
    ```

    Add the following lines to your `Podfile`:

    ```sh
    pod 'VirgilCrypto', '~> 5.1.0'
    pod 'VirgilCryptoPythia', '~> 0.10.0'
    ```

    Make sure you have `use_frameworks!` there as well. Then run `pod install` from inside the `ios` directory.

    **RN=0.60.x**

    Unfortunately, this library cannot be used with CocoaPods and React Native 0.60 until [this PR](https://github.com/facebook/react-native/pull/25619) makes it into a release (which will probably be a 0.61.0 release). The recommended option is to install the native dependency with Carthage and link the bridge (i.e. this library) manually.

2. **Carthage**
    With [Carthage](https://github.com/Carthage/Carthage) add the following line to your `Cartfile`:

    ```
    github "VirgilSecurity/virgil-crypto-x" ~> 5.1.0
    ```

    Then run `carthage update --platform iOS` from inside the `ios` folder.

    On your application target's “General” settings tab, in the “Linked Frameworks and Libraries” section, add following frameworks from the *Carthage/Build* folder inside your project's folder:

      - VirgilCrypto
      - VirgilCryptoFoundation
      - VSCCommon
      - VSCFoundation

    On your application target's “Build Phases” settings tab, click the “+” icon and choose “New Run Script Phase”. Create a Run Script in which you specify your shell (ex: */bin/sh*), add the following contents to the script area below the shell:

    ```sh
    /usr/local/bin/carthage copy-frameworks
    ```

    and add the paths to the frameworks you want to use under “Input Files”, e.g.:

    ```
    $(SRCROOT)/Carthage/Build/iOS/VirgilCrypto.framework
    $(SRCROOT)/Carthage/Build/iOS/VirgilCryptoFoundation.framework
    $(SRCROOT)/Carthage/Build/iOS/VirgilCryptoPythia.framework
    $(SRCROOT)/Carthage/Build/iOS/VSCCommon.framework
    $(SRCROOT)/Carthage/Build/iOS/VSCFoundation.framework
    $(SRCROOT)/Carthage/Build/iOS/VSCPythia.framework
    ```

### Android Setup

> Important! You will have to set `minSdkVersion` to at least `21` in your project's `build.gradle` file to use this library.

**RN<0.60**

```sh
react-native link react-native-virgil-crypto
```

With React Native 0.60 and later, linking is done automatically.

### Manual installation


#### iOS

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-virgil-crypto` and add `RNVirgilCrypto.xcodeproj`
3. In XCode, in the project navigator, select your project. Add `libRNVirgilCrypto.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
4. Run your project (`Cmd+R`)<

#### Android

1. Open up `android/app/src/main/java/[...]/MainApplication.java`
  - Add `import com.virgilsecurity.rn.crypto.RNVirgilCryptoPackage;` to the imports at the top of the file
  - Add `new RNVirgilCryptoPackage()` to the list returned by the `getPackages()` method
2. Append the following lines to `android/settings.gradle`:
  	```
  	include ':react-native-virgil-crypto'
  	project(':react-native-virgil-crypto').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-virgil-crypto/android')
  	```
3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
  	```
      compile project(':react-native-virgil-crypto')
  	```

## License
This library is released under the [3-clause BSD License](LICENSE).

## Support
Our developer support team is here to help you.

You can find us on [Twitter](https://twitter.com/VirgilSecurity) or send us email support@VirgilSecurity.com.

Also, get extra help from our support team on [Slack](https://virgilsecurity.com/join-community).