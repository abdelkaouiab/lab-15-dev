# 📱 Lab 15 — Gestion des Étudiants avec SQLite (Android Java)

---

## 📌 Aperçu

Ce TP consiste à développer une application Android simple permettant de gérer des étudiants en utilisant une base de données SQLite embarquée.

L'application est structurée en trois couches :

* Couche métier (modèle)
* Couche accès aux données (SQLite + CRUD)
* Couche présentation (Logcat + Interface utilisateur)

---

## 🎯 Objectifs

* Créer un modèle `Etudiant`
* Utiliser SQLite avec `SQLiteOpenHelper`
* Implémenter les opérations CRUD
* Tester avec Logcat
* Créer une interface simple (Ajouter / Chercher / Supprimer)

---

## ⚙️ Pré-requis

* Android Studio
* Java (bases)
* Activity + XML Layout

---

## 🏗️ Architecture

```
projet.fst.ma.app.classes
projet.fst.ma.app.util
projet.fst.ma.app.service
projet.fst.ma.app
```

---

# 🧩 1. Modèle Etudiant

```java
package projet.fst.ma.app.classes;

public class Etudiant {
    private int id;
    private String nom;
    private String prenom;

    public Etudiant(String nom, String prenom) {
        this.nom = nom;
        this.prenom = prenom;
    }

    public Etudiant() {}

    public int getId() { return id; }
    public void setId(int id) { this.id = id; }

    public String getNom() { return nom; }
    public void setNom(String nom) { this.nom = nom; }

    public String getPrenom() { return prenom; }
    public void setPrenom(String prenom) { this.prenom = prenom; }
}
```

---

# 🗄️ 2. SQLite Helper

```java
package projet.fst.ma.app.util;

import android.content.Context;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

public class MySQLiteHelper extends SQLiteOpenHelper {

    private static final int DATABASE_VERSION = 1;
    private static final String DATABASE_NAME = "ecole";

    private static final String CREATE_TABLE_ETUDIANT =
            "create table etudiant(" +
                    "id INTEGER primary key autoincrement," +
                    "nom TEXT," +
                    "prenom TEXT)";

    public MySQLiteHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL(CREATE_TABLE_ETUDIANT);
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP table if exists etudiant");
        onCreate(db);
    }
}
```

---

# 🔁 3. Service CRUD

```java
package projet.fst.ma.app.service;

import android.content.ContentValues;
import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.util.Log;

import java.util.ArrayList;
import java.util.List;

import projet.fst.ma.app.classes.Etudiant;
import projet.fst.ma.app.util.MySQLiteHelper;

public class EtudiantService {

    private static final String TABLE_NAME = "etudiant";
    private static final String KEY_ID = "id";
    private static final String KEY_NOM = "nom";
    private static final String KEY_PRENOM = "prenom";

    private static final String[] COLUMNS = {KEY_ID, KEY_NOM, KEY_PRENOM};

    private MySQLiteHelper helper;

    public EtudiantService(Context context) {
        helper = new MySQLiteHelper(context);
    }

    public void create(Etudiant e) {
        SQLiteDatabase db = helper.getWritableDatabase();
        ContentValues values = new ContentValues();
        values.put(KEY_NOM, e.getNom());
        values.put(KEY_PRENOM, e.getPrenom());
        db.insert(TABLE_NAME, null, values);
        db.close();
    }

    public Etudiant findById(int id) {
        SQLiteDatabase db = helper.getReadableDatabase();
        Cursor c = db.query(TABLE_NAME, COLUMNS, "id=?", new String[]{id+""}, null, null, null);

        if (c.moveToFirst()) {
            Etudiant e = new Etudiant();
            e.setId(c.getInt(0));
            e.setNom(c.getString(1));
            e.setPrenom(c.getString(2));
            c.close();
            db.close();
            return e;
        }

        c.close();
        db.close();
        return null;
    }

    public void delete(Etudiant e) {
        SQLiteDatabase db = helper.getWritableDatabase();
        db.delete(TABLE_NAME, "id=?", new String[]{e.getId()+""});
        db.close();
    }

    public List<Etudiant> findAll() {
        List<Etudiant> list = new ArrayList<>();
        SQLiteDatabase db = helper.getReadableDatabase();
        Cursor c = db.rawQuery("select * from " + TABLE_NAME, null);

        if (c.moveToFirst()) {
            do {
                Etudiant e = new Etudiant();
                e.setId(c.getInt(0));
                e.setNom(c.getString(1));
                e.setPrenom(c.getString(2));
                list.add(e);
            } while (c.moveToNext());
        }

        c.close();
        db.close();
        return list;
    }
}
```

---

# 🧪 4. Test Logcat

```java
EtudiantService es = new EtudiantService(this);

es.create(new Etudiant("ALAMI","ALI"));
es.create(new Etudiant("RAMI","AMAL"));

for(Etudiant e : es.findAll()){
    Log.d("ETUDIANT", e.getNom());
}
```

---

# 🎨 5. Interface XML

```xml
<LinearLayout ... >

    <EditText android:id="@+id/nom" />
    <EditText android:id="@+id/prenom" />
    <Button android:id="@+id/bn" android:text="Valider"/>

    <EditText android:id="@+id/id" />
    <Button android:id="@+id/load" android:text="Chercher"/>

    <TextView android:id="@+id/res" />

    <Button android:id="@+id/delete" android:text="Supprimer"/>

</LinearLayout>
```

---

# ⚡ 6. MainActivity (UI)

```java
add.setOnClickListener(v -> {
    es.create(new Etudiant(nom.getText().toString(), prenom.getText().toString()));
    Toast.makeText(this,"Ajouté",Toast.LENGTH_SHORT).show();
});

rechercher.setOnClickListener(v -> {
    Etudiant e = es.findById(Integer.parseInt(id.getText().toString()));
    if(e != null) res.setText(e.getNom()+" "+e.getPrenom());
});

supprimer.setOnClickListener(v -> {
    Etudiant e = es.findById(Integer.parseInt(id.getText().toString()));
    if(e != null) es.delete(e);
});
```

---

## ✅ Vérification

* Ajouter un étudiant → Toast affiché
* Chercher → Nom + prénom affichés
* Supprimer → Étudiant supprimé

---

## 🏁 Conclusion

Ce TP permet de comprendre comment créer une application Android avec SQLite en architecture simple (MVC), en manipulant des données locales avec CRUD complet.
