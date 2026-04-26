# lab5_sec

OWASP UnCrackable Level 2 (Lab 5)
📌 Introduction

Ce challenge consiste à extraire un secret caché dans l’application UnCrackable-Level2.apk.

Contrairement au Level 1, la logique de validation n’est plus visible en Java :
elle est déplacée en code natif (C/C++) via JNI (Java Native Interface), ce qui complique l’analyse statique classique.

📱 1. Analyse du Manifest
🧾 Informations clés
Élément	Valeur
📦 Package	owasp.mstg.uncrackable2
🚀 Main Activity	sg.vantagepoint.uncrackable2.MainActivity
📊 targetSdkVersion	28
📉 minSdkVersion	19
🔍 Observations
L’application démarre via MainActivity
Présence d’un intent-filter classique :
android.intent.action.MAIN
android.intent.category.LAUNCHER

👉 Cela confirme le point d’entrée principal

🧠 2. Analyse de MainActivity
🔎 Fonction de vérification

La méthode clé est :

verify(View view)
⚙️ Fonctionnement
📥 Récupération de l’entrée utilisateur depuis un EditText
🔄 Transmission à :
this.m.a(string);
🧩 Point important
m est une instance de CodeCheck
La méthode a() est déclarée native
public native boolean a(String str);

👉 Cela signifie que la logique n’est pas en Java, mais dans une librairie native.

🧬 3. Chargement du code natif (JNI)
⚠️ Élément critique
static {
    System.loadLibrary("foo");
}
📌 Explication
Charge la librairie native : libfoo.so
Contient la vraie logique de validation

👉 Impossible de récupérer le secret uniquement avec JADX ❌
👉 Il faut analyser le binaire natif ✅

🧪 4. Analyse de libfoo.so
🛠️ Outil utilisé
IDA Pro / Ghidra
🔍 Résultat de la décompilation
💡 Découverte clé

Une chaîne de référence est présente :

"Thanks for all the fish"
⚙️ Logique observée
Utilisation de :
strncmp(user_input, reference, 0x17)
📏 Détail important
0x17 = 23 caractères
Correspond exactement à la longueur du secret
✅ Condition de succès
Si la comparaison retourne 0
Et que certaines vérifications (anti-debug) passent

👉 La fonction retourne true

🔑 5. Secret découvert
🎯 Résultat final
Thanks for all the fish
🧪 Validation
✔️ Étapes
Entrer le secret dans l’application
L’app appelle la fonction native
strncmp retourne succès
🎉 Résultat
Success! This is the correct secret.
🔄 Flux global
Utilisateur input
        ↓
MainActivity.verify()
        ↓
CodeCheck.a(string)  (JNI)
        ↓
libfoo.so
        ↓
strncmp(input, "Thanks for all the fish", 23)
        ↓
✅ Success / ❌ Fail
🧠 Conclusion

Ce niveau illustre une technique classique de protection :

🔒 Déplacement de la logique critique en natif
🎯 Objectifs :
Rendre l’analyse plus difficile
Éviter la récupération directe via JADX
⚠️ Limites de cette approche
Le secret reste présent en clair dans le binaire
Les outils comme IDA ou Ghidra permettent de le récupérer
🛠️ Compétences mobilisées
Reverse engineering Android
Analyse JNI
Analyse binaire (C/C++)
Utilisation de IDA / Ghidra
🚀 Leçon importante

👉 Sécurité par obscurité ≠ sécurité réelle

Même en natif :

Les secrets peuvent être extraits
Le reverse engineering reste possible
