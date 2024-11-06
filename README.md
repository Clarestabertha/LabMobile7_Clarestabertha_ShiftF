# Tugas 6
Nama : Claresta Berthalita Jatmika

NIM : H1D022050

Shift Baru: F

### Cara Kerja Login
#### 1. Pengguna mengisi formulir login
Pada halaman login terdapat formulir yang harus diisi oleh pengguna yaitu berisi 'username' dan 'password'. Tata letak halaman ini diatur di file 'login.page.html'. Dibawah formulir login terdapat tombol 'Login' yang berguna untuk mengirim validasi ke dalam sistem. Tombol login ini memicu fungsi 'login()' di file 'login.page.ts' saat di klik.
```html
<ion-row>
    <ion-col>
      <ion-button type="submit" color="primary" expand="block" (click)="login()">Login</ion-button>
    </ion-col>
  </ion-row>
```

### 2. Fungsi login pada LoginPage
Saat tombol ditekan akan memanggil fungsi 'login()' pada file 'login.page.ts' yang berguna untuk memeriksa apakan username dan password sudah diisi. Jika belum maka aplikasi akan memunculkan notifikasi dengan menggunakan 'AuthenticationService' dengan pesan 'Username atau Password Tidak Boleh Kosong'. Tetapi jika sudah diisi semua formulirnya maka data akan dikirim ke server menggunakan 'postMethod' dengan endpoint '/login.php'.
```ts
login() {
    if (this.username != null && this.password != null) {
      const data = {
        username: this.username,
        password: this.password
      }
      this.authService.postMethod(data, 'login.php').subscribe({
        next: (res) => {
          if (res.status_login == "berhasil") {
            this.authService.saveData(res.token, res.username);
            this.username = '';
            this.password = '';
            this.router.navigateByUrl('/home');
          } else {
            this.authService.notifikasi('Username atau Password Salah');
          }
        },
        error: (e) => {
          this.authService.notifikasi('Login Gagal Periksa Koneksi Internet Anda');
        }
      })
    } else {
      this.authService.notifikasi('Username atau Password Tidak Boleh Kosong');
    }
  }
```

### 3. Mengirim data ke server
Fungsi 'postMethod' pada 'AuthenticationService' menggunakan 'HttpClient' untuk mengirim data username dan password. Selanjutnya backend akan memproses login, apabila valid server akan mengembalikan respons yang berisi token autentikasi dan nama pengguna.
``` php
<?php
require 'koneksi.php';
$input = file_get_contents('php://input');
$data = json_decode($input, true);
$pesan = [];
$username = trim($data['username']);
$password = md5(trim($data['password']));
$query = mysqli_query($con, "select * from user where username='$username' and
password='$password'");
$jumlah = mysqli_num_rows($query);
if ($jumlah != 0) {
    $value = mysqli_fetch_object($query);
    $pesan['username'] = $value->username;
    $pesan['token'] = time() . '_' . $value->password;
    $pesan['status_login'] = 'berhasil';
} else {
    $pesan['status_login'] = 'gagal';
}
echo json_encode($pesan);
echo mysqli_error($con);
```

### 4. Menyimpan token dan status login
Apabila login berhasil sistem akan menyimpan token dan user di 'Capicator Preferences' menggunakan kunci 'TOKEN_KEY' dan 'USER_KEY'. Lalu sistem akan memperbarui status autentikasi dengan mengubah 'isAuthenticated' menjadi true dan menandakan bahwa pengguna sudah login. Selanjutnya sistem akan mengarahkan ke halaman utama home (/home) menggunakan router 'this.router.navigateByUrl('/home')'
```ts
saveData(token: string, user: any) {
    Preferences.set({ key: TOKEN_KEY, value: token });
    Preferences.set({ key: USER_KEY, value: user });
    this.token = token;
    this.nama = user;
    this.isAuthenticated.next(true);
  }
```

### 5. Proteksi rute dengan auth guard
Ketika pengguna mencoba mengakses halaman yang dilindungi seperti /home, 'authGuard' akan memeriksa status 'authenticationState'. Jika 'true' akses ke halaman akan diperbolehkan tetapi jika 'false' maka 'authGuard' akan kembali mengarahkan ke halaman login
```ts
return authService.authenticationState.pipe(
    filter((val) => val !== null),
    take(1),
    map((isAuthenticated) => {
      if (isAuthenticated) {
        return true;
      } else {
        router.navigateByUrl('/login', { replaceUrl: true });
        return true;
      }
    })
  );
```

### 6. Memuat Data token di awal
Saat sistem dijalankan 'loadData()' di 'AuthenticationService' dipanggil untuk mengambil token dan user yang tersimpan di 'Capicator Preferences' serta memperbarui status login, jika token dan user ditemukan maka 'isAuthenticated' diatur menjadi 'true' dan pengguna dianggap masih login.
```ts
async loadData() {
    const token = await Preferences.get({ key: TOKEN_KEY });
    const user = await Preferences.get({ key: USER_KEY });
    if (token && token.value && user && user.value) {
      this.token = token.value;
      this.nama = user.value;
      this.isAuthenticated.next(true);
    } else {
      this.isAuthenticated.next(false);
    }
  }
```

### 7. Logout
Pada halaman homepage yang strukturnya diatur di dalam file 'home.page.html' terdapat sebuah tombol logout yang akan memanggil fungsi 'logout()' dari file 'home.page.ts'. Jika pengguna menekan tombol logout maka sistem akan menghapus data autentikasi dengan fungsi 'clearData()' untuk menghapus 'TOKEN_KEY' dan 'USER_KEY'. Selanjutnya akan mengubah status autentikasi dengan mengubah 'isAuthenticated' menjadi 'false'. Terakhir sistem akan mengarahkan ke halaman login lagi dengan menggunakan 'this.router.navigateByUrl('/login');'
```ts
logout() {
    this.authService.logout();
    this.router.navigateByUrl('/login');
  }
```

## Hasil Aplikasi
![Screenshot Login](ss/halaman%20login.png)
![Screenshot Homepage](ss/halaman%20home.png)
