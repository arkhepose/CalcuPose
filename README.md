# CalcuPose
import json

# Emisyon faktörleri (IPCC & DEFRA 2024)
EMISYON_FAKTORLERI = {
    "yakıt": {
        "dizel": 2.68,  # kg CO₂e/litre (IPCC)
        "benzin": 2.31,
        "dogalgaz": 2.02,  # kg CO₂e/m3
    },
    "elektrik": {
        "turkiye": 0.442,  # kg CO₂e/kWh (Türkiye 2022)
        "abd": 0.4,  # kg CO₂e/kWh (ABD ortalaması)
        "ab": 0.25,  # kg CO₂e/kWh (AB ortalaması)
    },
    "ulaşım": {
        "otomobil": 0.2,  # kg CO₂e/km
        "otobus": 0.1,
        "tren": 0.05,
        "ucak": 0.255,
    },
    "tarım": {
        "buyukbas": 70.0,  # kg CH₄/yıl (inek başına)
        "kucukbas": 8.0,
        "pirinc": 1.5,  # kg CH₄/kg ürün
    },
    "atık": {
        "katı_atık": 1.2,  # kg CO₂e/kg çöp
        "biyogaz": -0.5,  # kg CO₂e/kg (negatif, çünkü emisyon azaltıcı)
    },
}

def yakit_emisyonu(yakit_tipi, miktar):
    """Yakıt tüketimine bağlı karbon emisyonlarını hesaplar."""
    return miktar * EMISYON_FAKTORLERI["yakıt"].get(yakit_tipi, 0)

def elektrik_emisyonu(tuketim, ulke):
    """Elektrik tüketimine bağlı karbon emisyonlarını hesaplar."""
    return tuketim * EMISYON_FAKTORLERI["elektrik"].get(ulke, 0)

def ulasim_emisyonu(ulasim_tipi, mesafe):
    """Ulaşım kaynaklı karbon emisyonlarını hesaplar."""
    return mesafe * EMISYON_FAKTORLERI["ulaşım"].get(ulasim_tipi, 0)

def tarim_emisyonu(tarim_turu, miktar):
    """Tarım kaynaklı emisyonları hesaplar."""
    return miktar * EMISYON_FAKTORLERI["tarım"].get(tarim_turu, 0)

def atik_emisyonu(atik_turu, miktar):
    """Atık yönetimi kaynaklı karbon emisyonlarını hesaplar."""
    return miktar * EMISYON_FAKTORLERI["atık"].get(atik_turu, 0)

def toplam_karbon_ayak_izi(veriler):
    """Girilen tüm verileri kullanarak toplam karbon ayak izini hesaplar."""
    toplam_emisyon = 0
    toplam_emisyon += yakit_emisyonu(veriler["yakit"]["tip"], veriler["yakit"]["miktar"])
    toplam_emisyon += elektrik_emisyonu(veriler["elektrik"]["tuketim"], veriler["elektrik"]["ulke"])
    toplam_emisyon += ulasim_emisyonu(veriler["ulasim"]["tip"], veriler["ulasim"]["mesafe"])
    toplam_emisyon += tarim_emisyonu(veriler["tarim"]["tip"], veriler["tarim"]["miktar"])
    toplam_emisyon += atik_emisyonu(veriler["atik"]["tip"], veriler["atik"]["miktar"])
    return toplam_emisyon

# Kullanıcıdan veri al
veri_ornegi = {
    "yakit": {"tip": "dizel", "miktar": 100},
    "elektrik": {"tuketim": 500, "ulke": "turkiye"},
    "ulasim": {"tip": "otomobil", "mesafe": 150},
    "tarim": {"tip": "buyukbas", "miktar": 10},
    "atik": {"tip": "katı_atık", "miktar": 50},
}

# Sonuçları yazdır
toplam_emisyon = toplam_karbon_ayak_izi(veri_ornegi)
print(f"Toplam Karbon Ayak İzi: {toplam_emisyon:.2f} kg CO₂e")

from flask import Flask, request, render_template, jsonify
from carbon_calculator import toplam_karbon_ayak_izi

app = Flask(__name__)

@app.route('/')
def index():
    """Ana sayfa (HTML arayüzü)"""
    return render_template("index.html")

@app.route('/hesapla', methods=['POST'])
def hesapla():
    """JSON verisini alır ve karbon ayak izi hesaplar"""
    veri = request.json
    sonuc = toplam_karbon_ayak_izi(veri)
    return jsonify({"Toplam Karbon Ayak İzi (kg CO₂e)": sonuc})

if __name__ == '__main__':
    app.run(debug=True)
import json

EMISYON_FAKTORLERI = {
    "yakıt": {"dizel": 2.68, "benzin": 2.31, "dogalgaz": 2.02},
    "elektrik": {"turkiye": 0.442, "abd": 0.4, "ab": 0.25},
    "ulaşım": {"otomobil": 0.2, "otobus": 0.1, "tren": 0.05, "ucak": 0.255},
    "tarım": {"buyukbas": 70.0, "kucukbas": 8.0, "pirinc": 1.5},
    "atık": {"katı_atık": 1.2, "biyogaz": -0.5},
}

def hesapla_emisyon(kategori, tip, miktar):
    return miktar * EMISYON_FAKTORLERI[kategori].get(tip, 0)

def toplam_karbon_ayak_izi(veriler):
    toplam = 0
    toplam += hesapla_emisyon("yakıt", veriler["yakit"]["tip"], veriler["yakit"]["miktar"])
    toplam += hesapla_emisyon("elektrik", veriler["elektrik"]["ulke"], veriler["elektrik"]["tuketim"])
    toplam += hesapla_emisyon("ulaşım", veriler["ulasim"]["tip"], veriler["ulasim"]["mesafe"])
    toplam += hesapla_emisyon("tarım", veriler["tarim"]["tip"], veriler["tarim"]["miktar"])
    toplam += hesapla_emisyon("atık", veriler["atik"]["tip"], veriler["atik"]["miktar"])
    return toplam
<!DOCTYPE html>
<html lang="tr">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Karbon Ayak İzi Hesaplayıcı</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
    <script defer src="{{ url_for('static', filename='script.js') }}"></script>
</head>
<body>
    <div class="container">
        <h1>Karbon Ayak İzi Hesaplayıcı</h1>
        <form id="carbonForm">
            <label>Yakıt Tipi:</label>
            <select id="yakitTipi">
                <option value="dizel">Dizel</option>
                <option value="benzin">Benzin</option>
            </select>
            <input type="number" id="yakitMiktar" placeholder="Litre" required>

            <label>Elektrik Tüketimi (kWh):</label>
            <input type="number" id="elektrikTuketim" required>
            
            <label>Ülke:</label>
            <select id="elektrikUlke">
                <option value="turkiye">Türkiye</option>
                <option value="abd">ABD</option>
            </select>

            <label>Ulaşım Tipi:</label>
            <select id="ulasimTipi">
                <option value="otomobil">Otomobil</option>
                <option value="ucak">Uçak</option>
            </select>
            <input type="number" id="ulasimMesafe" placeholder="KM" required>

            <button type="submit">Hesapla</button>
        </form>

        <h2>Toplam Karbon Ayak İzi: <span id="sonuc">-</span> kg CO₂e</h2>
    </div>
</body>
</html>
body {
    font-family: Arial, sans-serif;
    background-color: #f4f4f4;
    text-align: center;
}
.container {
    width: 50%;
    margin: auto;
    background: white;
    padding: 20px;
    box-shadow: 0px 0px 10px gray;
}
h1 {
    color: #333;
}
form {
    display: flex;
    flex-direction: column;
    gap: 10px;
}
input, select, button {
    padding: 10px;
    font-size: 16px;
}
button {
    background: green;
    color: white;
    border: none;
    cursor: pointer;
}
button:hover {
    background: darkgreen;
}
document.getElementById("carbonForm").addEventListener("submit", function(event) {
    event.preventDefault();

    let veri = {
        yakit: {
            tip: document.getElementById("yakitTipi").value,
            miktar: parseFloat(document.getElementById("yakitMiktar").value)
        },
        elektrik: {
            tuketim: parseFloat(document.getElementById("elektrikTuketim").value),
            ulke: document.getElementById("elektrikUlke").value
        },
        ulasim: {
            tip: document.getElementById("ulasimTipi").value,
            mesafe: parseFloat(document.getElementById("ulasimMesafe").value)
        },
        tarim: {"tip": "buyukbas", "miktar": 0},
        atik: {"tip": "katı_atık", "miktar": 0}
    };

    fetch("/hesapla", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(veri)
    })
    .then(response => response.json())
    .then(data => {
        document.getElementById("sonuc").innerText = data["Toplam Karbon Ayak İzi (kg CO₂e)"];
    });
});
