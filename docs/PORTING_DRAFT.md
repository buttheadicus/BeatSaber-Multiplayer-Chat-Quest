# Beat Saber Multiplayer Chat - Quest port (draft)

**Canonical doc for the port.** Edit wire format, checkpoints, and open questions **here**, in **`BeatSaber-Multiplayer-Chat-Quest`**. Behavior and bytecode-level truth come from upstream [Multiplayer Chat on PC](https://github.com/buttheadicus/BeatSaber-Multiplayer-Chat) (`MultiplayerChat` project: BSIPA, C#, Multiplayer Extensions).

**Status:** Native skeleton from **quest-mod-template** is in repo (`qpm.json`, `CMakeLists.txt`, `src/`). Run **`qpm restore`** and **`qpm s build`** when your NDK and codegen stack are ready. Transport hooks on Quest are TBD.

---

## 1. PC reference stack (what you are mirroring in behavior)

- **Bootstrap:** IPA plugin in `Plugin.cs`; Zenject bindings for lobby-scoped chat.
- **Custom packets:** `MultiplayerCore.Networking.Abstractions.MpPacket`, serialized with **LiteNetLib** (`NetDataWriter` / `NetDataReader`). Registration is via **`MpPacketSerializer`** from **Multiplayer Extensions** (`ChatManager.RegisterPacketCallbacks`, `ModPresenceManager` for presence only).
- **Discriminators:** Each `MpPacket` subtype gets a runtime packet id inside Multiplayer Extensions / Multiplayer Core. Those **numeric ids are not defined in this repository** and can change with ME versions. Any Quest-side stack must either use a compatible registrar or negotiate a compatible framing layer. Payload layout **below the discriminator** must match LiteNetLib order so bytes match once the envelope is peeled.

---

## 2. Session encryption (plaintext before `Encrypted*` / `VoiceMessage` payloads)

Implemented in upstream PC **`Core/EncryptionManager.cs`** (see PC repo). Lobby members derive the same key.

| Item | Value |
|------|--------|
| Cipher | AES-256, **CBC**, **PKCS7** padding |
| IV | **16** random bytes, prepended |
| MAC | **HMAC-SHA256**, **32** bytes, over **concat(IV, ciphertext)** |
| On-wire blob | `IV (16)` + **ciphertext** + `HMAC (32)` |
| Key derivation | `Rfc2898DeriveBytes` (PBKDF2): password = UTF-8 of **sorted comma-separated platform user ids** (same sort as PC lobby list), salt = UTF-8 `MultiplayerChat.v1`, **10000** iterations, **SHA-256**, **32** byte key |

**Decrypt:** Split IV, ciphertext (length = total - 48); verify HMAC over IV||ciphertext; then AES-CBC decrypt with PKCS7.

Voice and chat both wrap **plaintext** with this envelope before placing bytes in **`PutBytesWithLength`** fields below (unless noted otherwise).

---

## 3. LiteNetLib field order (`Serialize` defines wire order)

Strings use **`Put(string)` / `GetString()`** (same rules as LiteNetLib). **`PutBytesWithLength`** writes a length prefix then raw bytes.

### 3.1 `EncryptedChatPacket`

| Order | Field | Notes |
|------|--------|------|
| 1 | Payload | `EncryptedPayload`; max **4096** bytes after decrypt for acceptance on receive (oversized rejects) |
| 2 | TargetUserId | Empty = broadcast chat; **both** `TargetUserId` and **`TargetChatId`** non-empty = DM routing (see `ChatPacketIdValidation`) |
| 3 | NameColor | 6-char hex without `#`; optional display color for sender |
| 4 | SenderChatId | **8-digit** persistent id (required for 0.2.0 rules on PC) |
| 5 | TargetChatId | DM recipient Chat ID |

**Deserialize** tolerates truncated streams (older peers): if payload absent or empty, following fields cleared.

---

### 3.2 `VoiceMessagePacket`

Plaintext inside encryption is **`VoiceMessageCodec`** binary (**VMSG**, see section 5), then encrypted same as section 2.

| Order | EncryptedPayload, TargetUserId, NameColor, SenderChatId, TargetChatId |

Max encrypted blob checked on PC against **`MaxPlaintextVoiceBytes` + slack** (~4 MiB plaintext class); see source for exact cap.

---

### 3.3 `ModPresencePacket`

Not encrypted.

| Order | Field |
|-------|--------|
| 1 | TargetUserId (empty = broadcast presence) |
| 2 | `IsIgnoredFromSong`, **byte** 0 or 1 |
| 3 | SenderChatId |
| 4 | SenderNameColor |
| 5 | `IsSlzCompanionClient`, **byte** 0 or 1 |

**Deserialize:** Presence byte may be absent on very old wire (backward compat path in PC).

---

### 3.4 `ChatActivityPacket`

| Order | Activity **byte**, SenderChatId, SenderNameColor |

**Activity constants:** `TypingStart = 1`, `TypingStop = 2`, `RecordingVoiceStart = 3`, `RecordingVoiceStop = 4`.

---

### 3.5 `DmIntroNotifyPacket`

| TargetUserId, SenderNameColor, SenderChatId, TargetChatId |

### 3.6 `MuteNotifyPacket`

| TargetUserId, IsMuted (byte), SenderNameColor, SenderChatId, TargetChatId |

### 3.7 `DmStoppedNotifyPacket`

| TargetUserId, SenderNameColor, SenderChatId, TargetChatId |

### 3.8 `TalkToNotifyPacket`

| TargetUserId, IsStopped (byte), SenderNameColor, SenderChatId, TargetChatId, AlsoTalkingToOthersCsv |

### 3.9 `ListenToNotifyPacket`

| TargetUserId, IsStopped (byte), SenderNameColor, SenderChatId, TargetChatId, AlsoListeningToOthersCsv |

### 3.10 `VoiceDeafenStatePacket`

| IsDeaf (byte), SenderChatId, SenderNameColor |

---

## 4. Persistent Chat ID (0.2.0)

- Format: **8-digit** decimal string (**10_000_000** to **99_999_999**), see **`ChatPersistentId.cs`** on PC.
- On PC, stored under user data (**DPAPI**). **Quest must define** an equivalent stable store per device/account for parity (so DM routing sees the same semantic id behavior).

Routing rules (`global` vs `DM`) are summarized in **`ChatPacketIdValidation`** on upstream PC (`Core/ChatPacketIdValidation.cs`).

---

## 5. Voice plaintext (`VoiceMessageCodec`, before AES)

| Offset | Size | Meaning |
|--------|------|---------|
| 0 | 4 | ASCII `VMSG` |
| 4 | 1 | Version (currently **1**) |
| 5 | 4 | Sample rate (**int32**, `BitConverter` / little-endian) |
| 9 | 2 | Channels (**ushort**) |
| 11 | 4 | Frame count (**int32**) |
| 15 | rest | Interleaved **float32** samples, little-endian |

---

## 6. Porting checklist (rough order)

1. Quest mod loads (skeleton in repo; **`qpm restore`** + **`qpm s build`** produce `libmultiplayerchat.so`, then **`qpm s qmod`** when NDK and codegen are configured).
2. Identify **multiplayer packet API** on Quest (same logical lobby + custom messages toward PC parity).
3. Implement **EncryptionManager-equivalent** and **EncryptedChatPacket** serde; verify interoperability with PC in a controlled lobby test.
4. **ModPresencePacket** plus remaining packet types needed for chosen feature slice.
5. Voice path: **VoiceMessageCodec** + capture/playback hooks on Quest.
6. UI last (different stack than BSML/HMUI).

---

## 7. Open questions (fill as you decide)

- [ ] Exact **Quest multiplayer / Beat Together** API for registering custom payloads equivalent to MpPacket.
- [ ] Whether **numeric packet ids** can be aligned with PC Multiplayer Extensions or need a versioned side channel.
- [ ] **Quest Chat ID** persistence path (file path, permissions, backup).
- [ ] Cross-play testing matrix: PC BS version vs Quest game version.

---

*Last draft pass: serialized layout matches upstream PC repo paths `Network/*.cs`, `Core/EncryptionManager.cs`, `Core/VoiceMessageCodec.cs`, `Core/ChatPacketIdValidation.cs`.*
