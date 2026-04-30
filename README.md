# LAB 10 — Guide d'installation et d'utilisation de Frida
**Cours : Sécurité des applications mobiles**  


---

##  Table des matières

1. [Objectifs du lab](#objectifs)
2. [Environnement de travail](#environnement)
3. [Étape 1 — Installation de Python et Frida](#étape-1)
4. [Étape 2 — Installation d'ADB](#étape-2)
5. [Étape 3 — Déploiement de frida-server](#étape-3)
6. [Étape 4 — Test de connexion](#étape-4)
7. [Étape 5 — Injection minimale](#étape-5)
8. [Étape 6 — Console interactive Frida](#étape-6)
9. [Étape 7 — Bibliothèques de chiffrement et hooks réseau](#étape-7)
10. [Étape 8 — Hooks Java](#étape-8)
11. [Conclusion](#conclusion)

---

##  Objectifs du lab <a name="objectifs"></a>

- Installer et configurer Frida (client Python + CLI) sur Windows
- Déployer et lancer `frida-server` sur un émulateur Android
- Établir une connexion et injecter des scripts d'instrumentation
- Observer les bibliothèques natives, les appels réseau et les accès fichiers
- Hooker des méthodes Java liées à SharedPreferences, SQLite et au débogage

---

##  Environnement de travail <a name="environnement"></a>

| Composant | Valeur |
|-----------|--------|
| Système d'exploitation | Windows 11 (10.0.26200) |
| Architecture PC | AMD64 |
| Python | 3.14.4 |
| Frida client | 17.9.3 |
| frida-server | 17.9.3 |
| ADB | 1.0.41 (version 37.0.0) |
| Émulateur | Android Emulator 5554 |
| Image système | Google APIs Intel x86_64 — Android 14.0 (API 34) |
| Architecture émulateur | x86_64 |

---

## Étape 1 — Installation de Python et Frida <a name="étape-1"></a>

### 1.1 Vérification de Python

```powershell
python --version
```

> **Résultat obtenu :**
> ```
> Python 3.14.4
> ```

### 1.2 Installation de Frida

```powershell
pip install --upgrade frida frida-tools
```

### 1.3 Vérification de Frida

```powershell
frida --version
```
<img width="960" height="442" alt="Capture d&#39;écran 2026-04-30 085512" src="https://github.com/user-attachments/assets/aab7a64c-adb9-4ab3-ab66-5f124337a7b5" />


---

## Étape 2 — Installation d'ADB <a name="étape-2"></a>

### 2.1 Vérification d'ADB

```powershell
adb version
```
<img width="957" height="195" alt="Capture d&#39;écran 2026-04-30 090127" src="https://github.com/user-attachments/assets/2b52b974-6368-4383-aeef-df17e47f6bf7" />


### 2.2 Détection de l'émulateur

```powershell
adb devices
```

<img width="944" height="299" alt="Capture d&#39;écran 2026-04-30 090451" src="https://github.com/user-attachments/assets/cb6c3ac4-c517-40e5-b50d-458071c7c275" />

---

## Étape 3 — Déploiement de frida-server <a name="étape-3"></a>

### 3.1 Identification de l'architecture

```powershell
adb shell getprop ro.product.cpu.abi
```

> **Résultat obtenu :**
> ```
> x86_64
> ```

### 3.2 Téléchargement de frida-server

Fichier téléchargé : `frida-server-17.9.3-android-x86_64.xz`  
Source : https://github.com/frida/frida/releases/tag/17.9.3

> ⚠️ La version de `frida-server` doit correspondre exactement à la version du client Frida.

### 3.3 Décompression

Extraction avec **7-Zip** → fichier renommé en `frida-server`

### 3.4 Copie vers l'émulateur

```powershell
adb push C:\Users\a\Downloads\frida-server /data/local/tmp/
```
<img width="943" height="104" alt="Capture d&#39;écran 2026-04-30 191651" src="https://github.com/user-attachments/assets/e608a6fb-9f10-41b5-a11e-f9ca7856028e" />


### 3.5 Attribution des droits d'exécution

```powershell
adb shell chmod 755 /data/local/tmp/frida-server
```

### 3.6 Lancement de frida-server

```powershell
adb shell "/data/local/tmp/frida-server &"
```

### 3.7 Vérification

```powershell
adb shell ps | findstr frida
```
<img width="960" height="146" alt="Capture d&#39;écran 2026-04-30 191701" src="https://github.com/user-attachments/assets/0e3d9d5c-b250-4496-834d-924a17b4e701" />


---

## Étape 4 — Test de connexion <a name="étape-4"></a>

```powershell
frida-ps -U
```
<img width="964" height="809" alt="Capture d&#39;écran 2026-04-30 191957" src="https://github.com/user-attachments/assets/1b2b9455-2942-4485-bbaa-6f82e5f57a27" />

---

## Étape 5 — Injection minimale <a name="étape-5"></a>

### 5.1 Test Java — hello.js

**Contenu du fichier `hello.js` :**

```javascript
Java.perform(function () {
  console.log("[+] Frida Java.perform OK");
});
```

**Commande d'injection :**

```powershell
frida -U -n com.android.systemui -l C:\Users\a\Downloads\hello.js
```
<img width="951" height="570" alt="Capture d&#39;écran 2026-04-30 224811" src="https://github.com/user-attachments/assets/100c7213-9290-4c20-b04c-ec74bdbd67a3" />
`

✅ Confirme que Frida peut instrumenter le runtime Java de l'application.

### 5.2 Test natif — hello_native.js

**Contenu du fichier `hello_native.js` :**

```javascript
console.log("[+] Script chargé");

Interceptor.attach(Module.getGlobalExportByName("recv"), {
  onEnter(args) {
    console.log("[+] recv appelée");
  }
});
```

**Commande d'injection :**

```powershell
frida -U -n com.android.systemui -l C:\Users\a\Downloads\hello_native.js
```
<img width="951" height="564" alt="Capture d&#39;écran 2026-04-30 225220" src="https://github.com/user-attachments/assets/665bbb9f-a76a-404e-a521-f92c5f3dd1ea" />


✅ Confirme que Frida peut intercepter des fonctions natives.

---

## Étape 6 — Console interactive Frida <a name="étape-6"></a>

**Commande de lancement :**

```powershell
frida -U -n com.android.systemui -l C:\Users\a\Downloads\hello.js
```

### Commandes exécutées dans la console

| Commande | Résultat obtenu |
|----------|----------------|
| `Process.arch` | `"x64"` |
| `Process.mainModule` | `app_process64` — `/system/bin/app_process64` |
| `Process.getModuleByName("libc.so")` | base: `0x7176fa040000`, size: 1212416 |
| `Process.getModuleByName("libc.so").getExportByName("recv")` | `0x7176fa0cbf50` |
| `Process.getModuleByName("libc.so").getExportByName("connect")` | `0x7176fa0b6910` |
| `Process.getModuleByName("libc.so").getExportByName("send")` | `0x7176fa0cc9d0` |
| `Java.available` | `true` |
| `Process.id` | `959` |
| `Process.platform` | `"linux"` |

<img width="919" height="809" alt="Capture d&#39;écran 2026-04-30 225356" src="https://github.com/user-attachments/assets/242b1b64-76fb-4ba4-9d9a-12ec423779ab" />

---

## Étape 7 — Bibliothèques de chiffrement et hooks réseau <a name="étape-7"></a>

### 7.1 Détection des bibliothèques de chiffrement

**Commande :**

```javascript
Process.enumerateModules().filter(m =>
  m.name.indexOf("ssl") !== -1 ||
  m.name.indexOf("crypto") !== -1 ||
  m.name.indexOf("boring") !== -1
)
```

**Résultat obtenu :**

| Bibliothèque | Chemin | Taille |
|-------------|--------|--------|
| `libcrypto.so` | `/system/lib64/` | 2 048 000 octets |
| `libjavacrypto.so` | `/apex/com.android.conscrypt/lib64/` | 262 144 octets |
| `libcrypto.so` | `/apex/com.android.conscrypt/lib64/` | 2 048 000 octets |
| `libssl.so` | `/apex/com.android.conscrypt/lib64/` | 655 360 octets |
| `libcrypto_httpengine.so` | `/apex/com.android.tethering/lib64/` | 1 114 112 octets |

✅ L'application utilise des bibliothèques TLS/SSL et de cryptographie natives.

### 7.2 Hook sur connect — hook_connect.js

```javascript
console.log("[+] Hook connect chargé");

const connectPtr = Process.getModuleByName("libc.so").getExportByName("connect");
console.log("[+] connect trouvée à : " + connectPtr);

Interceptor.attach(connectPtr, {
  onEnter(args) {
    console.log("[+] connect appelée");
    console.log("    fd = " + args[0]);
    console.log("    sockaddr = " + args[1]);
  },
  onLeave(retval) {
    console.log("    retour = " + retval.toInt32());
  }
});
```
<img width="906" height="334" alt="Capture d&#39;écran 2026-04-30 225811" src="https://github.com/user-attachments/assets/318bdb56-d132-454f-81c2-1e3ed7edc153" />


### 7.3 Hook sur send et recv — hook_network.js

```javascript
console.log("[+] Hooks réseau chargés");

const sendPtr = Process.getModuleByName("libc.so").getExportByName("send");
const recvPtr = Process.getModuleByName("libc.so").getExportByName("recv");

console.log("[+] send trouvée à : " + sendPtr);
console.log("[+] recv trouvée à : " + recvPtr);

Interceptor.attach(sendPtr, {
  onEnter(args) {
    console.log("[+] send appelée");
    console.log("    fd = " + args[0]);
    console.log("    len = " + args[2].toInt32());
  }
});

Interceptor.attach(recvPtr, {
  onEnter(args) {
    console.log("[+] recv appelée");
    console.log("    fd = " + args[0]);
    console.log("    len demandé = " + args[2].toInt32());
  },
  onLeave(retval) {
    console.log("    recv retourne = " + retval.toInt32());
  }
});
```

<img width="929" height="375" alt="Capture d&#39;écran 2026-04-30 225853" src="https://github.com/user-attachments/assets/2ba0ad49-64b7-434f-87d8-94e2e9478471" />


### 7.4 Hook sur open et read — hook_file.js

```javascript
console.log("[+] Hook fichiers chargé");

const openPtr = Process.getModuleByName("libc.so").getExportByName("open");
const readPtr = Process.getModuleByName("libc.so").getExportByName("read");

console.log("[+] open trouvée à : " + openPtr);
console.log("[+] read trouvée à : " + readPtr);

Interceptor.attach(openPtr, {
  onEnter(args) {
    this.path = args[0].readUtf8String();
    console.log("[+] open appelée : " + this.path);
  }
});

Interceptor.attach(readPtr, {
  onEnter(args) {
    console.log("[+] read appelée");
    console.log("    fd = " + args[0]);
    console.log("    taille = " + args[2].toInt32());
  }
});
```
<img width="905" height="652" alt="Capture d&#39;écran 2026-04-30 230001" src="https://github.com/user-attachments/assets/ccf6a09f-9183-4a2b-892c-e9119008ded6" />

✅ Des appels `read` actifs ont été détectés en temps réel.

---

## Étape 8 — Hooks Java <a name="étape-8"></a>

### 8.1 Hook SharedPreferences — hook_prefs.js

```javascript
Java.perform(function () {
  console.log("[+] Hook SharedPreferences chargé");

  var Impl = Java.use("android.app.SharedPreferencesImpl");

  Impl.getString.overload("java.lang.String", "java.lang.String").implementation = function (key, defValue) {
    var result = this.getString(key, defValue);
    console.log("[SharedPreferences][getString] key=" + key + " => " + result);
    return result;
  };

  Impl.getBoolean.overload("java.lang.String", "boolean").implementation = function (key, defValue) {
    var result = this.getBoolean(key, defValue);
    console.log("[SharedPreferences][getBoolean] key=" + key + " => " + result);
    return result;
  };
});
```

<img width="885" height="320" alt="Capture d&#39;écran 2026-04-30 230043" src="https://github.com/user-attachments/assets/bd11dfec-dc52-46d9-8e20-31cbe62fd8be" />


### 8.2 Hook SQLite — hook_sqlite.js

```javascript
Java.perform(function () {
  console.log("[+] Hook SQLite chargé");

  var SQLiteDatabase = Java.use("android.database.sqlite.SQLiteDatabase");

  SQLiteDatabase.execSQL.overload("java.lang.String").implementation = function (sql) {
    console.log("[SQLite][execSQL] " + sql);
    return this.execSQL(sql);
  };

  SQLiteDatabase.rawQuery.overload("java.lang.String", "[Ljava.lang.String;").implementation = function (sql, args) {
    console.log("[SQLite][rawQuery] " + sql);
    return this.rawQuery(sql, args);
  };
});
```

<img width="916" height="298" alt="Capture d&#39;écran 2026-04-30 230121" src="https://github.com/user-attachments/assets/49853d27-8b85-4c74-a45f-1e20faa93996" />


### 8.3 Hook Debug — hook_debug.js

```javascript
Java.perform(function () {
  console.log("[+] Hook Debug chargé");

  var Debug = Java.use("android.os.Debug");

  Debug.isDebuggerConnected.implementation = function () {
    var result = this.isDebuggerConnected();
    console.log("[Debug] isDebuggerConnected() => " + result);
    return result;
  };

  Debug.waitingForDebugger.implementation = function () {
    var result = this.waitingForDebugger();
    console.log("[Debug] waitingForDebugger() => " + result);
    return result;
  };
});
```

<img width="870" height="314" alt="Capture d&#39;écran 2026-04-30 230230" src="https://github.com/user-attachments/assets/2877cf2a-f0c7-4053-973e-ce232f6c2887" />

### 8.4 Hook Runtime.exec — hook_runtime.js

```javascript
Java.perform(function () {
  console.log("[+] Hook Runtime.exec chargé");

  var Runtime = Java.use("java.lang.Runtime");

  Runtime.exec.overload("java.lang.String").implementation = function (cmd) {
    console.log("[Runtime.exec] " + cmd);
    return this.exec(cmd);
  };
});
```

<img width="876" height="317" alt="image" src="https://github.com/user-attachments/assets/a5142371-5d59-4452-89f6-1c33dce37cb5" />


### 8.5 Hook File Java — hook_file_java.js

```javascript
Java.perform(function () {
  console.log("[+] Hook File chargé");

  var File = Java.use("java.io.File");

  File.$init.overload("java.lang.String").implementation = function (path) {
    console.log("[File] nouveau chemin : " + path);
    return this.$init(path);
  };
});
```

<img width="878" height="309" alt="Capture d&#39;écran 2026-04-30 230440" src="https://github.com/user-attachments/assets/b4fac955-748f-4846-9183-b8719502b470" />


##  Conclusion <a name="conclusion"></a>

Ce lab a permis de mettre en place un environnement complet d'analyse dynamique avec Frida sur Android. Les compétences acquises sont les suivantes :

- Installation et configuration de Frida sur Windows
- Déploiement de `frida-server` sur un émulateur Android (x86_64)
- Utilisation de la console interactive pour inspecter un processus Android
- Identification des bibliothèques natives (libc, libssl, libcrypto)
- Interception des fonctions réseau (`connect`, `send`, `recv`) et fichiers (`open`, `read`)
- Hooking de méthodes Java sensibles (SharedPreferences, SQLite, Debug, Runtime)

Ces techniques constituent une base solide pour l'analyse de sécurité des applications mobiles Android dans un cadre éthique et pédagogique.

---

> ⚠️ **Avertissement éthique** : Frida doit être utilisé uniquement sur des appareils et applications dont vous avez les droits d'analyse. Toute utilisation non autorisée est illégale.

---

