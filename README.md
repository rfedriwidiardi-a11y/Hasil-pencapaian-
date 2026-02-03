# Hasil-pencapaian-
function updateData() {
  let t = document.getElementById("tabel");
  let notif = [];
  let karyawanArr = [];

  // Ambil semua data dari tabel
  for (let i = 1; i < t.rows.length; i++) {
    let nama = t.rows[i].cells[0].children[0].value.trim();
    let email = t.rows[i].cells[1].children[0].value.trim();
    let target = parseFloat(t.rows[i].cells[2].children[0].value.replace(',', '.')) || 0;
    let output = parseFloat(t.rows[i].cells[3].children[0].value.replace(',', '.')) || 0;
    let persen = target > 0 ? ((output / target) * 100).toFixed(1) : 0;

    karyawanArr.push({ nama, email, target, output, persen });
  }

  // Urutkan dari persen terbesar ke terkecil
  karyawanArr.sort((a, b) => b.persen - a.persen);

  // Hapus semua baris lama (kecuali header)
  while (t.rows.length > 1) t.deleteRow(1);

  // Tambahkan kembali baris yang sudah diurutkan
  karyawanArr.forEach((karyawan, index) => {
    let r = t.insertRow();
    let cells = [];
    for (let i = 0; i < 5; i++) {
      cells.push(r.insertCell());
    }

    // Nama
    let inputNama = document.createElement("input");
    inputNama.value = karyawan.nama;
    cells[0].appendChild(inputNama);

    // Email
    let inputEmail = document.createElement("input");
    inputEmail.value = karyawan.email;
    cells[1].appendChild(inputEmail);

    // Target
    let inputTarget = document.createElement("input");
    inputTarget.value = karyawan.target;
    cells[2].appendChild(inputTarget);

    // Output
    let inputOutput = document.createElement("input");
    inputOutput.value = karyawan.output;
    cells[3].appendChild(inputOutput);

    // Persen
    cells[4].innerText = karyawan.persen + "%";

    // Notifikasi & kirim email
    if (karyawan.persen <= 50 && karyawan.email) {
      notif.push(`${karyawan.nama || "Karyawan " + (index+1)} (${karyawan.email}) di bawah 50%`);

      emailjs.send('service_jfp4hw8', 'template_pgfl63e', {
        nama: karyawan.nama,
        persen: karyawan.persen,
        to_email: karyawan.email
      })
      .then(function(response) {
        console.log('Email terkirim ke ' + karyawan.email, response.status, response.text);
      }, function(error) {
        console.error('Gagal mengirim email ke ' + karyawan.email, error);
      });
    }
  });

  // Simpan data ke LocalStorage
  localStorage.setItem('karyawanData', JSON.stringify(karyawanArr));

  // Tampilkan grafik
  let labels = karyawanArr.map(k => k.nama || "Karyawan");
  let values = karyawanArr.map(k => k.persen);
  tampilGrafik(labels, values);

  // Tampilkan notifikasi
  document.getElementById("notif").innerHTML =
    notif.length ? "<b>Notifikasi:</b><br>" + notif.join("<br>") :
    "<b>✅ Semua karyawan di atas 50%</b>";
}
