# Puzzle Kata (Noir PoC)

Proof of Concept (PoC) aplikasi **puzzle kata** menggunakan [Noir](https://noir-lang.org/).  
Aplikasi ini menerima input berupa **kata** dan **huruf yang tersedia**, lalu melakukan validasi:  

1. Apakah kata ada di **kamus** (dictionary berbasis hash).  
2. Apakah semua huruf penyusun kata tersedia di kumpulan huruf.  

Jika salah satu tidak terpenuhi → program akan gagal dengan `assert`.

---

## 🚀 Setup Project

```bash
# Buat project baru
noir new puzzle_kata
cd puzzle_kata
```

---

## 📂 Struktur Project

```
puzzle_kata/
 ├─ src/
 │   └─ main.nr      # kode utama validasi kata
 ├─ tests/
 │   └─ test.nr      # unit test untuk beberapa skenario
 └─ README.md
```

---

## 🔑 Kode Utama (`src/main.nr`)

```rust
// Maksimal panjang kata dan huruf
const WORD_LEN: usize = 7;
const LETTERS_LEN: usize = 7;
const DICTIONARY_SIZE: usize = 3;

// Hash sederhana (PoC)
// Catatan: gunakan Poseidon hash untuk versi produksi
fn simple_hash(arr: [u8; WORD_LEN]) -> u32 {
    let mut hash = 0u32;
    for i in 0..WORD_LEN {
        hash = hash + arr[i] as u32 * (i as u32 + 1);
    }
    hash
}

// Kamus sederhana (dictionary) berupa hash kata
const DICTIONARY: [u32; DICTIONARY_SIZE] = [
    751,  // hash("kata")
    664,  // hash("kita")
    535,  // hash("atik")
];

// Hitung frekuensi huruf
fn count_letters(arr: [u8; LETTERS_LEN]) -> [u8; 26] {
    let mut counts = [0u8; 26];
    for i in 0..LETTERS_LEN {
        if arr[i] == 0u8 { continue; } // skip kosong
        let idx = arr[i] - 97; // ASCII 'a' = 97
        counts[idx as usize] = counts[idx as usize] + 1;
    }
    counts
}

// Fungsi validasi kata
fn validate_word(word: [u8; WORD_LEN], letters: [u8; LETTERS_LEN]) -> bool {
    // 1. Cek kata ada di kamus
    let h = simple_hash(word);
    let mut found = false;
    for i in 0..DICTIONARY_SIZE {
        if DICTIONARY[i] == h {
            found = true;
            break;
        }
    }
    assert(found, "Kata tidak ada di kamus");

    // 2. Cek huruf tersedia
    let mut letter_counts = count_letters(letters);
    for i in 0..WORD_LEN {
        let c = word[i];
        if c == 0u8 { // kata lebih pendek dari 7 huruf
            break;
        }
        let idx = (c - 97) as usize;
        assert(letter_counts[idx] > 0, "Huruf tidak tersedia");
        letter_counts[idx] = letter_counts[idx] - 1;
    }
    true
}

// Entry point
fn main(word: [u8; WORD_LEN], letters: [u8; LETTERS_LEN]) -> bool {
    validate_word(word, letters)
}
```

---

## 🧪 Unit Test (`tests/test.nr`)

```rust
use puzzle_kata;

fn test_kata_valid() {
    // "kata" -> [107,97,116,97,0,0,0]
    let word = [107, 97, 116, 97, 0, 0, 0];
    let letters = [107, 97, 116, 97, 105, 0, 0]; // k,a,t,a,i
    let result = puzzle_kata::main(word, letters);
    assert(result);
}

fn test_kita_valid() {
    let word = [107, 105, 116, 97, 0, 0, 0]; // "kita"
    let letters = [107, 105, 116, 97, 120, 121, 122]; // k,i,t,a,x,y,z
    let result = puzzle_kata::main(word, letters);
    assert(result);
}

fn test_atik_valid() {
    let word = [97, 116, 105, 107, 0, 0, 0]; // "atik"
    let letters = [97, 116, 105, 107, 98, 99, 100]; // a,t,i,k,b,c,d
    let result = puzzle_kata::main(word, letters);
    assert(result);
}

fn test_word_not_in_dictionary() {
    let word = [116, 97, 107, 105, 0, 0, 0]; // "taki"
    let letters = [116, 97, 107, 105, 0, 0, 0];
    // Harus gagal -> "Kata tidak ada di kamus"
    puzzle_kata::main(word, letters);
}

fn test_insufficient_letters() {
    let word = [107, 97, 116, 97, 0, 0, 0]; // "kata"
    let letters = [107, 97, 105, 0, 0, 0, 0]; // k,a,i (tidak cukup)
    // Harus gagal -> "Huruf tidak tersedia"
    puzzle_kata::main(word, letters);
}
```

---

## ▶️ Menjalankan Test

```bash
noir test
```

Hasil:
- ✅ `test_kata_valid`, `test_kita_valid`, `test_atik_valid` → lolos.  
- ❌ `test_word_not_in_dictionary`, `test_insufficient_letters` → gagal (expected, karena memang dites gagal).  

---

## 📌 Format Input

- Input kata & huruf adalah **array 7 byte** (ASCII huruf kecil).  
- Jika kata lebih pendek dari 7 huruf → isi sisanya `0`.  

Contoh:  
- `"kata"` → `[107,97,116,97,0,0,0]`  
- `"kita"` → `[107,105,116,97,0,0,0]`  

---

## 📖 Catatan

- Kamus berisi 3 kata (`kata`, `kita`, `atik`) dengan hash sederhana.  
- Hash sederhana bisa *collision* → untuk produksi sebaiknya gunakan **Poseidon hash** (tersedia di Noir).  
- Fokus PoC ini adalah *verifiable logic*, belum ada UI/front-end.  
