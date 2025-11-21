#!/bin/bash
# -----------------------------------------------
# Setup-Skript für GeoServer Proxy und PostGIS DB
# -----------------------------------------------

# --- Globale Variablen ---
DB_NAME="osgis"
DB_OWNER="user"
# ANPASSUNG: Pfad zur PBF-Datei
PBF_FILE_NAME="andorra-251105.osm.pbf"
PBF_PATH="database/data/$PBF_FILE_NAME"

# --- Funktion zur Überprüfung der Befehle ---
check_command() {
    if ! command -v "$1" &> /dev/null
    then
        echo "🚨 Fehler: Der Befehl '$1' konnte nicht gefunden werden."
        echo "Bitte stellen Sie sicher, dass $1 (z.B. node, npm, psql, osm2pgsql) installiert ist."
        exit 1
    fi
}

# --- 1. Node.js Proxy Einrichtung ---
echo "--- 📦 1. Node.js Proxy einrichten ---"
check_command "npm"

if [ -f "package.json" ]; then
    echo "Abhängigkeiten werden installiert..."
    npm install
    if [ $? -ne 0 ]; then
        echo "🚨 Fehler: NPM-Installation fehlgeschlagen."
        exit 1
    fi
    echo "✅ Node.js-Abhängigkeiten erfolgreich installiert."
else
    echo "⚠️ Warnung: 'package.json' nicht gefunden. Überspringe NPM-Installation."
fi

# --- 2. PostGIS Datenbank Einrichtung ---
echo -e "\n--- 💾 2. PostGIS-Datenbank einrichten ---"
check_command "psql"
check_command "osm2pgsql"

# Überprüfen, ob die PBF-Datei im erwarteten Pfad existiert
echo "Suche nach der PBF-Datei unter: $PBF_PATH"
if [ ! -f "$PBF_PATH" ]; then
    echo "🚨 Fehler: Die OSM-PBF-Datei '$PBF_FILE_NAME' wurde nicht gefunden."
    echo "Bitte stellen Sie sicher, dass sie im Ordner '$PBF_PATH' liegt."
    exit 1
fi

# Datenbank erstellen (mit Fehlerprüfung)
echo "Versuche, die Datenbank '$DB_NAME' zu erstellen..."
# Prüfe, ob die DB existiert, und erstelle sie nur, wenn sie nicht existiert
sudo -u postgres psql -c "SELECT 1 FROM pg_database WHERE datname = '$DB_NAME'" | grep -q 1 || \
sudo -u postgres psql -c "CREATE DATABASE $DB_NAME OWNER $DB_OWNER;"

if [ $? -eq 0 ]; then
    echo "Datenbank '$DB_NAME' existiert oder wurde erfolgreich erstellt."
else
    echo "🚨 Fehler beim Erstellen der Datenbank. Haben Sie die richtigen PostgreSQL-Berechtigungen?"
    exit 1
fi

# OSM-Daten laden
echo "Lade OSM-Daten ($PBF_PATH) in die Datenbank '$DB_NAME'..."
# ANPASSUNG: Der Pfad zur PBF-Datei wird an osm2pgsql übergeben
osm2pgsql -d $DB_NAME $PBF_PATH
if [ $? -eq 0 ]; then
    echo "✅ OSM-Daten erfolgreich geladen."
else
    echo "🚨 Fehler beim Laden der OSM-Daten mit osm2pgsql."
    exit 1
fi

# --- 3. Abschluss ---
echo -e "\n--- 🎉 Setup abgeschlossen ---"
echo "Sie können den Proxy nun starten:"
echo "node proxy.js"