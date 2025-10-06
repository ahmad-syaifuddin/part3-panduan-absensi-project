<div align="center">

<!-- <p> <b>Visitors Count 👁️</b> </p>
<img src="https://profile-counter.deno.dev/part3-panduan-absensi-project/count.svg" alt="Profile Counter Repo :: Visitor's Count" /> -->

</div>

# Part3 panduan absensi project

## saatnya kembali ke "markas" admin! 👮
### Setelah karyawan bisa absen, tugas admin adalah memonitor dan melihat laporannya. Kita akan membangun fitur Laporan Absensi Harian yang memungkinkan admin melihat data absensi semua karyawan dan memfilternya berdasarkan tanggal.

## Tahap 10: Membuat Laporan Absensi Harian (Admin)
🎯 Tujuan:
Membuat halaman laporan yang menampilkan tabel absensi harian semua karyawan, lengkap dengan filter tanggal.

Langkah 1: Membuat Controller Baru untuk Laporan
Agar kode tetap rapi dan tidak menumpuk di AttendanceController, kita akan buat Controller baru yang khusus menangani laporan untuk admin.

Jalankan perintah ini di terminal Anda:

```Bash
php artisan make:controller Admin/ReportController
```
Sekarang, buka file yang baru dibuat di app/Http/Controllers/Admin/ReportController.php dan isi dengan logika berikut:

```PHP
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;
use App\Models\Attendance;
use Carbon\Carbon;

class ReportController extends Controller
{
    /**
     * Menampilkan halaman laporan absensi harian.
     */
    public function dailyReport(Request $request)
    {
        // Tentukan tanggal. Jika ada input dari user, gunakan itu. Jika tidak, gunakan hari ini.
        $date = $request->input('date') ? Carbon::parse($request->input('date')) : Carbon::now('Asia/Makassar');

        // Ambil data absensi pada tanggal yang dipilih
        // Kita gunakan 'with' untuk mengambil data relasi 'employee' agar lebih efisien (Eager Loading)
        $attendances = Attendance::with('employee')
                                ->whereDate('date', $date)
                                ->orderBy('created_at', 'desc')
                                ->get();

        // Kirim data ke view
        return view('admin.reports.daily', [
            'attendances' => $attendances,
            'selectedDate' => $date->format('Y-m-d'), // Kirim tanggal yang dipilih ke view
        ]);
    }
}
```
Langkah 2: Membuat Rute Baru
Selanjutnya, kita buat alamat URL untuk mengakses halaman laporan ini.

Buka file routes/web.php dan tambahkan rute baru ini. Pastikan rute ini hanya bisa diakses oleh admin.

```PHP
// routes/web.php
use App\Http\Controllers\Admin\ReportController;

// ... rute lainnya

// Grup Rute untuk Admin
Route::middleware(['auth'])->group(function () { 
    // ... rute admin yang sudah ada ...
    Route::post('/admin/attendance/mark-alpa', [AttendanceController::class, 'markAlpa'])
            ->name('admin.attendance.mark_alpa');

    // RUTE BARU UNTUK LAPORAN HARIAN
    Route::get('/admin/reports/daily', [ReportController::class, 'dailyReport'])
            ->name('admin.reports.daily');
});
```

Langkah 3: Membuat View untuk Laporan
Sekarang kita buat tampilannya.

Buat folder baru: resources/views/admin.

Di dalam admin, buat folder lagi: reports.

Di dalam reports, buat file baru bernama daily.blade.php.

Isi file daily.blade.php dengan kode berikut:

```Blade

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

                    {{-- Filter Tanggal --}}
                    <div class="mb-6">
                        <form method="GET" action="{{ route('admin.reports.daily') }}">
                            <div class="flex items-center space-x-4">
                                <div>
                                    <label for="date" class="block text-sm font-medium text-gray-700">Pilih Tanggal:</label>
                                    <input type="date" name="date" id="date" value="{{ $selectedDate }}" class="mt-1 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md">
                                </div>
                                <div class="pt-5">
                                    <button type="submit" class="inline-flex items-center px-4 py-2 border border-transparent text-sm font-medium rounded-md shadow-sm text-white bg-indigo-600 hover:bg-indigo-700">
                                        Tampilkan Laporan
                                    </button>
                                </div>
                            </div>
                        </form>
                    </div>

                    {{-- Tabel Laporan --}}
                    <div class="overflow-x-auto">
                        <table class="min-w-full divide-y divide-gray-200">
                            <thead class="bg-gray-50">
                                <tr>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">No</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Nama Karyawan</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Jam Masuk</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Jam Pulang</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Status</th>
                                    <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Keterangan</th>
                                </tr>
                            </thead>
                            <tbody class="bg-white divide-y divide-gray-200">
                                @forelse ($attendances as $attendance)
                                    <tr>
                                        <td class="px-6 py-4">{{ $loop->iteration }}</td>
                                        <td class="px-6 py-4 font-medium text-gray-900">{{ $attendance->employee->nama_lengkap ?? 'Karyawan Tidak Ditemukan' }}</td>
                                        <td class="px-6 py-4">{{ $attendance->time_in ?? '--:--' }}</td>
                                        <td class="px-6 py-4">{{ $attendance->time_out ?? '--:--' }}</td>
                                        <td class="px-6 py-4">
                                            {{-- Logika Badge Status --}}
                                            @php
                                                $status = $attendance->status;
                                                $badgeColor = 'bg-gray-100 text-gray-800'; // Default
                                                if (str_contains($status, 'Hadir')) $badgeColor = 'bg-green-100 text-green-800';
                                                elseif (str_contains($status, 'Terlambat')) $badgeColor = 'bg-yellow-100 text-yellow-800';
                                                elseif (str_contains($status, 'Izin')) $badgeColor = 'bg-blue-100 text-blue-800';
                                                elseif (str_contains($status, 'Sakit')) $badgeColor = 'bg-orange-100 text-orange-800';
                                                elseif (str_contains($status, 'Alpa')) $badgeColor = 'bg-red-100 text-red-800';
                                            @endphp
                                            <span class="px-2 inline-flex text-xs leading-5 font-semibold rounded-full {{ $badgeColor }}">
                                                {{ $status }}
                                            </span>
                                        </td>
                                        <td class="px-6 py-4 text-sm text-gray-500">{{ $attendance->notes }}</td>
                                    </tr>
                                @empty
                                    <tr>
                                        <td colspan="6" class="px-6 py-4 text-center text-sm text-gray-500">
                                            Tidak ada data absensi untuk tanggal yang dipilih.
                                        </td>
                                    </tr>
                                @endforelse
                            </tbody>
                        </table>
                    </div>

                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
Langkah 4: Menambahkan Link Navigasi untuk Admin
Terakhir, agar admin bisa mengakses halaman ini, kita tambahkan link di menu navigasi.

Buka file resources/views/layouts/navigation.blade.php. Cari bagian menu untuk admin dan tambahkan link baru.

```blade
<div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
    <div class="flex justify-between h-16">
        <div class="flex">
            <div class="hidden space-x-8 sm:-my-px sm:ml-10 sm:flex">
                @if(Auth::user()->role === 'admin')
                    {{-- Navigasi untuk Admin --}}
                    <x-nav-link :href="route('dashboard')" :active="request()->routeIs('dashboard')">
                        {{ __('Dashboard') }}
                    </x-nav-link>
                    
                    {{-- LINK BARU UNTUK LAPORAN --}}
                    <x-nav-link :href="route('admin.reports.daily')" :active="request()->routeIs('admin.reports.daily')">
                        {{ __('Laporan Harian') }}
                    </x-nav-link>

                @elseif(Auth::user()->role === 'karyawan')
                    {{-- Navigasi untuk Karyawan --}}
                    @endif
            </div>
        </div>
        </div>
</div>
```
Jangan lupa untuk menambahkan link yang sama di bagian Responsive Navigation Menu agar muncul di tampilan mobile.

## ✅ Uji Coba
Login sebagai Admin.

Di menu navigasi atas, akan muncul link baru "Laporan Harian". Klik link tersebut.

Anda akan diarahkan ke halaman laporan yang secara default menampilkan data absensi untuk hari ini.

Gunakan filter tanggal untuk memilih hari kemarin atau tanggal lain, lalu klik "Tampilkan Laporan".

Pastikan data yang muncul di tabel sudah sesuai dengan tanggal yang Anda pilih dan statusnya benar.


---

### Next lanjut Part 4 untuk [Panduannya](https://github.com/ahmad-syaifuddin/part4-panduan-absensi-project.git).
