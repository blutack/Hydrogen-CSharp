<p align="center"> 
  <img src="https://i.imgur.com/VsCHqUk.png" alt="alt logo">
</p>

[![PayPal](https://drive.google.com/uc?id=1OQrtNBVJehNVxgPf6T6yX1wIysz1ElLR)](https://www.paypal.me/nxrighthere) [![Bountysource](https://drive.google.com/uc?id=19QRobscL8Ir2RL489IbVjcw3fULfWS_Q)](https://salt.bountysource.com/checkout/amount?team=nxrighthere) [![Discord](https://discordapp.com/api/guilds/515987760281288707/embed.png)](https://discord.gg/ceaWXVw)

This repository provides a managed C# wrapper for [Hydrogen](https://github.com/jedisct1/libhydrogen) cryptographic library which is created and maintained by [Frank Denis](https://github.com/jedisct1). You will need to [build](https://github.com/jedisct1/libhydrogen/wiki/Installation#downloading-the-source-code) the native library before you get started.

Usage
--------
Before starting to work, the library should be initialized using `Hydrogen.Library.Initialize();` function.

##### Generate random data:
```c#
// Unbounded
uint data = Hydrogen.Library.Random();

// Bounded
uint upperBound = 1000000;
uint data = Hydrogen.Library.Random(upperBound);
```

##### Declare a new context:
```c#
// Only the first 8 characters will be used
string context = "hydrocontext";
```

##### Generic hashing:
```c#
string message = "Arbitrary data to hash";
byte[] data = Encoding.ASCII.GetBytes(message);;
int hashLength = 16;
byte[] hash = new byte[hashLength]; // Storage for hash

// Without a key
if (Hydrogen.Library.Hash(hash, hashLength, data, data.Length, context))
	Console.WriteLine("Hash successfully generated!");
  
// With a key
byte[] hashKey = new byte[Hydrogen.Library.hashKeyBytes];

Hydrogen.Library.HashKeygen(hashKey);

if (Hydrogen.Library.Hash(hash, hashLength, data, data.Length, context, hashKey))
	Console.WriteLine("Hash successfully generated using key!");
```

##### Password hashing:
```c#
string password = "feelsgoodman";
byte[] masterKey = new byte[Hydrogen.Library.masterKeyBytes];
int keyLength = 32;
byte[] key = new byte[keyLength]; // Storage for high-entropy key

// Generate master key
Hydrogen.Library.MasterKeygen(masterKey);

// Generate high-entropy key from a password using master key
if (Hydrogen.Library.DeterministicKey(key, keyLength, password, password.Length, context, masterKey, 1000, 1024, 1))
	Console.WriteLine("High-entropy key successfully generated!");

// Generate authenticated representative of the password to store in the database
byte[] storedKey = new byte[Hydrogen.Library.storedBytes];

if (Hydrogen.Library.StorageKey(storedKey, password, password.Length, masterKey, 1000, 1024, 1))
	Console.WriteLine("Authenticated representative successfully generated!");

// Verify stored key
if (Hydrogen.Library.VerifyKey(storedKey, password, password.Length, masterKey, 1000, 1024, 1))
	Console.WriteLine("Stored key successfully verified!");
  
// Reencrypt stored key
byte[] newMasterKey = new byte[Hydrogen.Library.masterKeyBytes];

Hydrogen.Library.MasterKeygen(newMasterKey);

if (Hydrogen.Library.ReencryptKey(storedKey, masterKey, newMasterKey))
	Console.WriteLine("Stored key successfully reencrypted!");
  
// Upgrade stored key
if (Hydrogen.Library.UpgradeKey(storedKey, newMasterKey, 2000, 1024, 2))
	Console.WriteLine("Stored key successfully upgraded!");
```

##### Secret key encryption:
```c#
byte[] key = new byte[Hydrogen.Library.secretKeyBytes]; // Storage for secret key

// Generate secret key
Hydrogen.Library.SecretKeygen(key);

// Secret data
string message = "Secret message for future generations";
byte[] data = Encoding.ASCII.GetBytes(message);

// Encrypt data
byte[] cipher = new byte[data.Length + Hydrogen.Library.headerBytes];

if (Hydrogen.Library.Encrypt(cipher, data, data.Length, context, key))
	Console.WriteLine("Data successfully encrypted!");

// Create probe for cipher
byte[] probe = new byte[Hydrogen.Library.probeBytes];

if (Hydrogen.Library.CreateProbe(probe, cipher, cipher.Length, context, key))
	Console.WriteLine("Probe successfully created!");
	
// Verify probe
if (Hydrogen.Library.VerifyProbe(probe, cipher, cipher.Length, context, key))
	Console.WriteLine("Probe successfully verified!");

// Decrypt data
byte[] data = new byte[cipher.Length - Hydrogen.Library.headerBytes];

if (Hydrogen.Library.Decrypt(data, cipher, cipher.Length, context, key))
	Console.WriteLine("Data successfully decrypted!");
```

##### Secure network communication based on Noise protocol:
```c#
// Server
KeyPair serverKeyPair = default(KeyPair);

// Generate long-term key pair
Hydrogen.Library.ExchangeKeygen(out serverKeyPair);

/* Send `serverKeyPair.publicKey` to the client */

// Client
SessionKeyPair clientSessionKeyPair = default(SessionKeyPair);
byte[] packet = new byte[Hydrogen.Library.packetBytes];

// Generate session keys and a packet with an ephemeral public key
if (Hydrogen.Library.N1(out clientSessionKeyPair, packet, serverKeyPair.publicKey)))
	Console.WriteLine("Session key pair successfully generated!");

/* Send `packet` to the server */

// Server
SessionKeyPair serverSessionKeyPair = default(SessionKeyPair);

// Process the initial request from the client and generate session keys
if (Hydrogen.Library.N2(out serverSessionKeyPair, packet, ref serverKeyPair))
	Console.WriteLine("Session key pair successfully generated!");

/* Send a signal to the client that secure communication is established */

// Client
string message = "Do you want to take a look at my high-poly things tonight?";
byte[] data = Encoding.ASCII.GetBytes(message);;
byte[] packet = new byte[data.Length + Hydrogen.Library.headerBytes];

// Encrypt data
if (Hydrogen.Library.Encrypt(packet, data, data.Length, context, clientSessionKeyPair.sendKey))
	Console.WriteLine("Data successfully encrypted!");

/* Send `packet` to the server */

// Server
byte[] data = new byte[packet.Length - Hydrogen.Library.headerBytes];

// Decrypt data
if (Hydrogen.Library.Decrypt(data, packet, packet.Length, context, serverSessionKeyPair.receiveKey))
	Console.WriteLine("Data successfully decrypted!");

Console.WriteLine("Received message: " + Encoding.ASCII.GetString(data));
```
