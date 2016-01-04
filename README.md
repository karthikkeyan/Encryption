#C Libraries in Swift 

Signing a string using CommonCrypto library. I found below code by googling, thanks to someone who written this piece of code,

```swift
import Foundation
import CommonCrypto

enum HMACAlgorithm {
    case MD5, SHA1, SHA224, SHA256, SHA384, SHA512
    
    func toCCEnum() -> CCHmacAlgorithm {
        var result: Int = 0
        switch self {
        case .MD5:
            result = kCCHmacAlgMD5
        case .SHA1:
            result = kCCHmacAlgSHA1
        case .SHA224:
            result = kCCHmacAlgSHA224
        case .SHA256:
            result = kCCHmacAlgSHA256
        case .SHA384:
            result = kCCHmacAlgSHA384
        case .SHA512:
            result = kCCHmacAlgSHA512
        }
        return CCHmacAlgorithm(result)
    }
    
    func digestLength() -> Int {
        var result: CInt = 0
        switch self {
        case .MD5:
            result = CC_MD5_DIGEST_LENGTH
        case .SHA1:
            result = CC_SHA1_DIGEST_LENGTH
        case .SHA224:
            result = CC_SHA224_DIGEST_LENGTH
        case .SHA256:
            result = CC_SHA256_DIGEST_LENGTH
        case .SHA384:
            result = CC_SHA384_DIGEST_LENGTH
        case .SHA512:
            result = CC_SHA512_DIGEST_LENGTH
        }
        return Int(result)
    }
}

extension String {
    
    func encrypt(algorithm: HMACAlgorithm, key: String) -> String {
        let str = self.cStringUsingEncoding(NSUTF8StringEncoding)
        let strLen = UInt(self.lengthOfBytesUsingEncoding(NSUTF8StringEncoding))
        let digestLen = algorithm.digestLength()
        let result = UnsafeMutablePointer<CUnsignedChar>.alloc(digestLen)
        let keyStr = key.cStringUsingEncoding(NSUTF8StringEncoding)
        let keyLen = UInt(key.lengthOfBytesUsingEncoding(NSUTF8StringEncoding))
        
        CCHmac(algorithm.toCCEnum(), keyStr!, Int(keyLen), str!, Int(strLen), result)
        let data = NSData(bytesNoCopy: result, length: digestLen)
        result.destroy()
        
        return data.base64EncodedStringWithOptions(NSDataBase64EncodingOptions())
    }
    
}
```

## Module Mapping 

Everything is fine so far, isn’t it? The trouble comes when you import **Common Crypto** library into you code. You will see the below error,

![alt text] (https://github.com/karthikkeyan/Encryption/blob/master/error.png "Common Crypto Error")

This is because swift compiler cannot find the **CommonCrypto** module because it is a **C** library. Again I googled for the solution and I found lots of answers which confused me even more. After a lot of trial-error I found a solution that works for me.

The solution is to help the swift compiler to find where the **CommonCrypto** library is located in your SDK path. In other words we need to map any **C** libraries to swift compiler.

Follow the steps below,

1. Create a directory inside your project directory. The name of the your newly created directory must be the library name that you wanna use, in this case it is **CommonCrypto**.
2. Create **module.map** file inside the directory you have created.
3. Add the below code inside **module.map** file,

```swift
module CommonCrypto [system] {
    header “/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/usr/include/CommonCrypto/CommonCrypto.h”
    link “CommonCrypto”
    export *
}
```

So you project directory would looks something like this,

![alt text] (https://github.com/karthikkeyan/Encryption/blob/master/filestructure.png "Module mapping directory structure")

> Note: You don’t have to add these files in you project. Just create a directory and a map file.


## Project Settings 

Now that you have your mapping files ready, we have to add that directory’s path, in **Import Paths** under **Search Paths** section in your **Build Settings**.

![alt text] (https://github.com/karthikkeyan/Encryption/blob/master/importpath.png "Add Import Path")

> Note: Since I have added **CommonCrypto** directory as a root level directory, I didn’t include the full path. Use **$SRCROOT** as a prefix if you created **CommonCrypto** directory in different location.

Finally added **$(SDKROOT)/usr/lib/system** in your **Library Search Paths** under **Search Paths** section in your **Build Settings**.

![alt text] (https://github.com/karthikkeyan/Encryption/blob/master/librarypath.png "Add Library Search Path")

Thats it, now your should be able to use CommonCrypto C Library in your swift project.

Sample Encryption Code,

```swift
print(“Hello World”.encrypt(HMACAlgorithm.MD5, key: “SecretKey123”))
```

## Summary

1. Create a directory with your library Name
2. Add module.map inside that directory
3. Add that directory in Import Paths
4. Add **$(SDKROOT)/usr/lib/system** in Library Search Paths
5. Build and Run

