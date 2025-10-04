# Part3 panduan absensi project

## saatnya kembali ke "markas" admin! 👮
### Setelah karyawan bisa absen, tugas admin adalah memonitor dan melihat laporannya. Kita akan membangun fitur Laporan Absensi Harian yang memungkinkan admin melihat data absensi semua karyawan dan memfilternya berdasarkan tanggal.

## Tahap 10: Membuat Laporan Absensi untuk Admin
###Langkah 27: Membuat ReportController
Kita buat controller baru yang khusus menangani semua hal terkait laporan agar kode tetap rapi.

Jalankan perintah ini di terminal:

```Bash
php artisan make:controller ReportController
```

### Langkah 28: Membuat Rute untuk Halaman Laporan
Selanjutnya, kita definisikan "alamat" untuk halaman laporan di dalam grup rute khusus admin.

Buka file routes/web.php.

Tambahkan rute berikut di dalam grup middleware(['auth', 'role:admin']).

```PHP
// routes/web.php
use App\Http\Controllers\ReportController; // <-- Jangan lupa tambahkan ini
// ...
Route::middleware(['auth', 'role:admin'])->group(function () {
    Route::resource('users', UserController::class);
    // RUTE UNTUK LAPORAN
    Route::get('/reports', [ReportController::class, 'index'])->name('reports.index'); // <-- TAMBAHKAN INI
});

// ...
```
### Langkah 29: Implementasi Logika Laporan di Controller
Sekarang kita isi "otak" dari fitur laporan di ReportController. Logika ini akan mengambil data absensi dan menangani filter tanggal.

Buka app/Http/Controllers/ReportController.php.

Isi dengan kode berikut:

```PHP

<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use App\Models\Attendance; // <-- Import model
use Carbon\Carbon;          // <-- Import Carbon

class ReportController extends Controller
{
    public function index(Request $request)
    {
        // Tentukan tanggal filter: dari input request atau default hari ini
        $filterDate = $request->input('date', Carbon::now('Asia/Makassar')->format('Y-m-d'));

        // Query data absensi
        $attendances = Attendance::with(['employee.user']) // Eager loading relasi bertingkat
                                 ->where('date', $filterDate)
                                 ->latest()
                                 ->paginate(15);

        // Kirim data ke view
        return view('reports.index', compact('attendances', 'filterDate'));
    }
}
```
Penjelasan Kode:

Request $request: Kita butuh ini untuk menangkap input filter dari pengguna.

$request->input('date', ...): Mengambil nilai dari input form dengan nama date. Jika tidak ada, maka akan menggunakan tanggal hari ini sebagai nilai default.

with(['employee.user']): Ini adalah nested eager loading. Kita memberitahu Eloquent: "Ambil semua data Attendance, sertakan juga data employee yang terhubung, DAN sertakan juga data user yang terhubung dengan employee tersebut." Ini sangat efisien untuk menampilkan nama karyawan dari tabel users.

Langkah 30: Membuat View Laporan
Kini kita buat halaman untuk menampilkan filter dan tabel laporannya.

Buat folder baru bernama reports di dalam resources/views.

Di dalam folder reports, buat file baru bernama index.blade.php.

Isi file index.blade.php dengan kode lengkap berikut:

```HTML
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Laporan Absensi Harian') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">

                    <form method="GET" action="{{ route('reports.index') }}">
                        <div class="flex items-center space-x-4 mb-6">
                            <div>
                                <x-input-label for="date" :value="__('Pilih Tanggal')" />
                                <x-text-input id="date" type="date" name="date" :value="$filterDate" />
                            </div>
                            <div class="pt-6">
                                <x-primary-button>
                                    <i class="fas fa-filter mr-2"></i> Filter
                                </x-primary-button>
                                <a href="{{ route('reports.index') }}" class="ml-2 inline-flex items-center px-4 py-2 bg-gray-300 border border-transparent rounded-md font-semibold text-xs text-gray-700 uppercase tracking-widest hover:bg-gray-400 focus:outline-none focus:border-gray-900 focus:ring ring-gray-300 disabled:opacity-25 transition ease-in-out duration-150">
                                    Reset
                                </a>
                            </div>
                        </div>
                    </form>

                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">No</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Nama Karyawan</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">NIP</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Jam Masuk</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Jam Pulang</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Status</th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                @forelse ($attendances as $attendance)
                                    <tr>
                                        <td class="px-6 py-4">{{ $loop->iteration + $attendances->firstItem() - 1 }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->employee->user->name }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->employee->nip }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_in }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">{{ $attendance->time_out ?? '-' }}</td>
                                        <td class="px-6 py-4 whitespace-nowrap">
                                            {{-- Logika badge status seperti di riwayat absensi --}}
                                            @if ($attendance->status == 'Hadir')
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-green-100 text-green-800">Hadir</span>
                                            @elseif ($attendance->status == 'Terlambat')
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-yellow-100 text-yellow-800">Terlambat</span>
                                            @else
                                                <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full bg-red-100 text-red-800">{{ $attendance->status }}</span>
                                            @endif
                                        </td>
                                    </tr>
                                @empty
                                    <tr>
                                        <td colspan="6" class="px-6 py-4 text-center text-gray-500">
                                            Tidak ada data absensi untuk tanggal ini.
                                        </td>
                                    </tr>
                                @endorselse
                            </tbody>
                        </table>
                    </div>

                    <div class="mt-4">
                        {{ $attendances->appends(request()->query())->links() }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
Catatan Penting:

{{ $attendances->appends(request()->query())->links() }}: Ini sangat krusial. Perintah appends(request()->query()) memastikan bahwa ketika admin berpindah halaman (misal dari halaman 1 ke 2), parameter filter tanggal di URL akan tetap terbawa. Tanpa ini, filter akan ter-reset setiap kali pindah halaman.

Langkah 31: Mengaktifkan Link Navigasi Admin
Terakhir, kita aktifkan link menu "Laporan Absensi" untuk admin.

Buka resources/views/layouts/navigation.blade.php.

Cari link "Laporan Absensi" dan aktifkan.

Ubah ini (desktop & mobile):

```HTML
<x-nav-link :href="'#'" {{-- :href="route('reports.index')" :active="request()->routeIs('reports.*')" --}}>
    {{ __('Laporan Absensi') }}
</x-nav-link>
```
Menjadi:

```HTML

<x-nav-link :href="route('reports.index')" :active="request()->routeIs('reports.*')">
    {{ __('Laporan Absensi') }}
</x-nav-link>
```
(Lakukan hal yang sama untuk <x-responsive-nav-link>)

Uji Coba
Login sebagai admin.

Klik menu "Laporan Absensi". Anda akan diarahkan ke halaman laporan.

Secara default, halaman akan menampilkan data absensi untuk hari ini.

Gunakan filter tanggal untuk memilih tanggal lain (misalnya kemarin atau lusa) lalu klik "Filter". Tabel akan diperbarui sesuai tanggal yang dipilih.

Klik tombol "Reset" untuk kembali menampilkan data hari ini.

Fitur laporan dasar untuk admin sekarang sudah selesai! Admin kini punya alat untuk memantau kehadiran karyawan setiap hari.
