# 🧠 Tugas Praktikum: Menyimpan Histori Kalkulator (DynamoDB)

## 💡 Apa itu DynamoDB?
DynamoDB adalah layanan database **NoSQL** dari AWS. Bayangkan jika Lambda adalah "otak" yang menghitung, maka **DynamoDB adalah "buku catatan"** untuk menyimpan hasil hitunganmu agar tidak hilang meski browser ditutup.

---

## 🎯 Tujuan Tugas
Murid dapat menghubungkan kode backend (Lambda) ke database cloud (DynamoDB) untuk menyimpan data secara permanen.

---

## 🚀 Langkah-Langkah Praktikum

### 1. Membuat Tabel Database (DynamoDB)
1. Buka dashboard **DynamoDB** di AWS Console.
2. Klik tombol **"Create table"**.
3. **Table name**: `histori-kalkulator`
4. **Partition key**: `id` (Pilih tipe `String`).
5. Scroll ke bawah dan klik **"Create table"**.

### 2. Memberikan Izin (Permissions)
Agar Lambda bisa menulis ke database, kita harus memberinya izin akses:
1. Buka dashboard **Lambda** -> klik fungsi `kalkulator-[nama-siswa]` Anda.
2. Klik tab **"Configuration"** -> **"Permissions"**.
3. Klik link pada bagian **"Role name"** (biasanya berwarna biru). Anda akan diarahkan ke halaman IAM.
4. Di halaman IAM, klik tombol **"Add permissions"** -> **"Attach policies"**.
5. Di kolom pencarian, ketik: `AmazonDynamoDBFullAccess`.
6. Centang kotak di samping kebijakan tersebut, lalu klik **"Attach policies"**.

### 3. Update Kode Program Lambda
1. Kembali ke halaman Lambda Anda, klik tab **"Code"**.
2. Hapus seluruh isi `lambda_function.py` dan ganti dengan kode lengkap di bawah ini:

```python
import json
import boto3
import uuid

# Inisialisasi koneksi ke DynamoDB
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('histori-kalkulator')

def lambda_handler(event, context):
    params = event.get('queryStringParameters', {})
    
    # 1. Bagian API: Jika ada angka, hitung dan simpan ke Database
    if 'a' in params and 'b' in params:
        try:
            a, b = int(params.get('a')), int(params.get('b'))
            hasil = a + b
            
            # Simpan ke DynamoDB
            table.put_item(Item={
                'id': str(uuid.uuid4()),
                'ekspresi': f"{a} + {b} = {hasil}"
            })
            
            return {
                'statusCode': 200, 
                'headers': {"Content-Type": "application/json"},
                'body': json.dumps({'hasil': hasil})
            }
        except:
            return {'statusCode': 400, 'body': 'Input harus berupa angka'}

    # 2. Bagian UI: Jika tidak ada angka, kirim Tampilan (UI) Kalkulator
    html_ui = """
    <!DOCTYPE html>
    <html>
    <body style="font-family:sans-serif; text-align:center; padding:50px;">
        <h2>Kalkulator Cloud + Histori</h2>
        <input type="number" id="a" placeholder="Angka 1"> + 
        <input type="number" id="b" placeholder="Angka 2">
        <button onclick="hitung()">Hitung & Simpan</button>
        <h3 id="hasil">Hasil: -</h3>
        <script>
            async function hitung() {
                const a = document.getElementById('a').value;
                const b = document.getElementById('b').value;
                const url = window.location.href + "?a=" + a + "&b=" + b;
                const res = await fetch(url).then(r => r.json());
                document.getElementById('hasil').innerText = "Hasil: " + res.hasil;
            }
        </script>
    </body>
    </html>
    """
    return {'statusCode': 200, 'headers': {"Content-Type": "text/html"}, 'body': html_ui}
```
3. Klik tombol **"Deploy"** (biru) untuk menyimpan.

### 4. Mengetes Hasil
1. Buka kembali **Function URL** Anda di browser.
2. Lakukan beberapa perhitungan (input angka dan klik "Hitung & Simpan").
3. Buka dashboard **DynamoDB** -> **Tables** -> klik `histori-kalkulator`.
4. Klik tombol **"Explore table items"**.
5. **Boom!** Anda akan melihat data hasil perhitungan Anda tersimpan di database.

---

## ⚠️ PENTING: Pembersihan (Wajib!)
Setelah selesai, **WAJIB** hapus sumber daya agar tidak ada biaya:
1. **DynamoDB**: Pilih tabel -> **Delete**.
2. **Lambda**: Pilih fungsi -> **Actions** -> **Delete function**.
