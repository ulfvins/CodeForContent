+++
author = "Richard Ulfvin"
date = "2018-10-19"
description = "AES in PowerShell"
linktitle = ""
title = "How to use AES Encryption in PowerShell"
type = "post"

+++

Advanced Encryption Standard (or for short AES, also known as Rijndael) is a powerful symmetric block cypher approved by NIST. It uses 128-bit blocks and keys at the size of 128, 192 and 256 bits. 
AES uses a transformation schema, where it first using a substitution table, then shifts data rows, then mixes columns to finally do a simple XOR operation on each column using different parts of the encryption key. The longer the key is, the more rounds it need to operate on. So, for a 128-bit key there are 10 rounds, 192-bit key 12 rounds and for 256-bit key 14 rounds.
AES has proven to be a reliable cipher, and the only practical successful attacks against AES have leveraged side-channel attacks on weaknesses found in the implementation or key management of specific AES-based encryption products.

>I am going to be presenting on how to use Key Vault at the MS Event TechDays in Sweden, so I figured that I try to use some crypto classes from .NET in PowerShell (for the sake of IT Pros usage). The demo on stage will be how to use AES in Hybrid Encryption scenario with Key Vault (not this post though).

So here goes: How to use AES encryption in PowerShell.

First in AES we need to setup the Cipher key we are going to use. The recommended way would be to use a well suitable randomizer to get a 256-bit key. For this we use the .NET Class *RNGCryptoServiceProvider*. We also need to randomize the Initialization Vector (16 bytes for the Pre-Round transformation). 

{{< highlight powershell >}}
#NOTE Create a Key and IV:
$RNG = New-Object System.Security.Cryptography.RNGCryptoServiceProvider

$AESEncryptionKey     = [System.Byte[]]::new(32) #NOTE: 32 Bytes (256-bit Key)
$RNG.GetBytes($AESEncryptionKey)

$InitializationVector = [System.Byte[]]::new(16) #NOTE: 16 Bytes (128-bit IV)
$RNG.GetBytes($InitializationVector)
{{< /highlight >}}

Next, we need to setup the AES Cipher. Here we use the .NET Class *AesCryptoServiceprovider*, and setup the IV and Key we are using for the encryption:

{{< highlight powershell >}}
#NOTE: Create a AES Crypto Provider:
$AESCipher = New-Object System.Security.Cryptography.AesCryptoServiceProvider

#NOTE: Add the Key and IV to the Cipher
$AESCipher.Key        = $AESEncryptionKey
$AESCipher.IV         = $InitializationVector
{{< /highlight >}}

Then we need to setup our secret, transform it to a byte array and append the IV with the data to be able to reuse it with decryption operations.

{{< highlight powershell >}}
#NOTE: Set the secret:
$MySecretText         = "This is my super secret!"

#NOTE: Encrypt data with AES:
$UnencryptedBytes     = [System.Text.Encoding]::UTF8.GetBytes($MySecretText)
$Encryptor            = $AESCipher.CreateEncryptor()
$EncryptedBytes       = $Encryptor.TransformFinalBlock($UnencryptedBytes, 0, $UnencryptedBytes.Length)

#NOTE: Save the IV information with the data:
[byte[]] $FullData    = $AESCipher.IV + $EncryptedBytes
{{< /highlight >}}

Last, we transform the byte array back to a string format suitable for storage. And dispose the Cipher and RNG objects.

{{< highlight powershell >}}
#NOTE: Transforms the data to string format
$CipherText           = [System.Convert]::ToBase64String($FullData)

#NOTE: Cleanup the Cipher and KeyGenerator
$AESCipher.Dispose()
$RNG.Dispose()
{{< /highlight >}}

To decrypt the data, we first initialize the AesCryptoServiceProvider and append the key. Then we reverse the process and transform the data back to a byte array. We get the IV from the first 16 bytes, then perform the decrypt operation on the byte array. Last we transform the byte array back to a string format.

{{< highlight powershell >}}
#NOTE: Decrypt data with AES:
$AESCipher = New-Object System.Security.Cryptography.AesCryptoServiceProvider

#NOTE: Set the AES Key:
$AESCipher.Key        = $AESEncryptionKey

#NOTE: Get the encrypted string:
$EncryptedBytes       = [System.Convert]::FromBase64String($CipherText)

#Note: Get the IV data for AES:
$AESCipher.IV  = $EncryptedBytes[0..15]

#NOTE: Decrypt the with AES:
$Decryptor            = $AESCipher.CreateDecryptor();
$UnencryptedBytes     = $Decryptor.TransformFinalBlock($EncryptedBytes, 16, $EncryptedBytes.Length - 16)

#NOTE: Transform the data from byte array to string
$MySecretText         = [System.Text.Encoding]::UTF8.GetString($UnencryptedBytes)

$AESCipher.Dispose()

#NOTE: Get the secret:
$MySecretText
{{< /highlight >}}

I hope this post was inspiring and you start to protect your data with a lite PowerShell magic!