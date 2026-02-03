# Hasil-pencapaian-
<!DOCTYPE html>
<html lang="id">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Monitoring Kinerja Karyawan</title>

<!-- Chart.js CDN -->
<script src="https://cdn.jsdelivr.net/npm/chart.js@4.3.0/dist/chart.umd.min.js"></script>
<!-- EmailJS CDN -->
<script src="https://cdn.jsdelivr.net/npm/emailjs-com@3/dist/email.min.js"></script>

<style>
body{
  font-family:Arial, sans-serif;
  background:linear-gradient(135deg,#1e3c72,#2a5298);
  color:#fff;
  padding:20px;
}
.container{
  max-width:1100px;
  margin:auto;
  background:#111;
  padding:20px;
  border-radius:12px;
  box-shadow:0 0 20px rgba(0,0,0,.6);
}
h1{text-align:center}
table{
  width:100%;
  border-collapse:collapse;
  margin-top:15px;
}
th,td{
  border:1px solid #444;
  padding:8px;
  text-align:center;
}
th{background:#222}
input{
  width:100%;
  padding:5px;
  background:#000;
  color:#fff;
  border:1px solid #555;
}
button{
  padding:10px 15px;
  border:none;
  border-radius:6px;
  background:#00c6ff;
  cursor:pointer;
  font-weight:bold;
}
button:hover{background:#00aaff}
.notif{
  margin-top:15px;
  background:#300;
  padding:10px;
  border-radius:8px;
}
canvas{margin-top:20px}
</style>
</head>

<body>
<div class="container">
<h1>Monitoring Kinerja Karyawan</h1>

<button onclick="tambahBaris()">➕ Tambah Karyawan</button>

<table id="tabel">
<tr>
<th>Nama</th>
<th>Email</th>
<th>Target</th>
<th>Output</th>
<th>%</th>
</tr>
</table>

<br>
<button onclick="updateData()">UPDATE & CEK</button>

<div id="notif" class="notif"></div>

<canvas id="grafik"></canvas>
</div>

<script>
let chart;

// Delay EmailJS init sampai halaman siap
window.addEventListener('DOMContentLoaded', () => {
  // Ganti dengan PUBLIC KEY EmailJS Anda
  try {
    emailjs.init('FXF4s7JlMUYeHI6j0'); 
  } catch(e) {
    console.warn("EmailJS belum dikonfigurasi. Email tidak akan terkirim.");
  }

  // Load data dari LocalStorage
  let data = localStorage.getItem('karyawanData');
  if(data){
    JSON.parse(data).forEach(k => tambahBaris(k));
  }
  updateData();
});

function tambahBaris(karyawan = null){
  let t=document.getElementById("tabel");
  let r=t.insertRow();
  for(let i=0;i<5;i++){
    let c=r.insertCell();
    if(i<4){
      let input = document.createElement("input");
      if(karyawan){
        if(i==0) input.value = karyawan.nama;
        if(i==1) input.value = karyawan.email;
        if(i==2) input.value = karyawan.target;
        if(i==3) input.value = karyawan.output;
      }
      c.appendChild(input);
    } else {
      c.innerText = karyawan ? karyawan.persen + "%" : "0%";
    }
  }
}

function updateData(){
  let t = document.getElementById("tabel");
  let labels = [], values = [];
  let notif = [];
  let karyawanArr = [];

  for(let i=1; i<t.rows.length; i++){
    let nama = t.rows[i].cells[0].children[0].value;
    let email = t.rows[i].cells[1].children[0].value;
    let target = parseFloat(t.rows[i].cells[2].children[0].value) || 0;
    let output = parseFloat(t.rows[i].cells[3].children[0].value) || 0;
    let persen = target ? ((output/target)*100).toFixed(1) : 0;

    t.rows[i].cells[4].innerText = persen + "%";
    labels.push(nama || "Karyawan " + i);
    values.push(persen);

    karyawanArr.push({nama, email, target, output, persen});

    if(persen <= 50 && email){
      notif.push(`⚠️ ${nama} (${email}) di bawah 50%`);

      // Cek EmailJS agar tidak crash kalau belum konfigurasi
      if(typeof emailjs !== 'undefined' && emailjs.send){
        emailjs.send('service_jfp4hw8', 'template_pgfl63e', {
          nama: nama,
          persen: persen,
          email: email
        }).then(
          ()=>console.log('Email terkirim ke ' + email),
          (err)=>console.error('Gagal kirim email ke ' + email, err)
        );
      }
    }
  }

  localStorage.setItem('karyawanData', JSON.stringify(karyawanArr));

  tampilGrafik(labels, values);

  document.getElementById("notif").innerHTML =
    notif.length ? "<b>Notifikasi:</b><br>" + notif.join("<br>") :
    "<b>✅ Semua karyawan di atas 50%</b>";
}

function tampilGrafik(l,v){
  if(chart) chart.destroy();
  chart=new Chart(document.getElementById("grafik"),{
    type:"bar",
    data:{
      labels:l,
      datasets:[{
        label:"Persentase Kinerja",
        data:v,
        backgroundColor:"rgba(0,198,255,0.6)"
      }]
    },
    options:{
      responsive:true,
      scales:{y:{beginAtZero:true,max:100}}
    }
  });
}
</script>
</body>
</html>
