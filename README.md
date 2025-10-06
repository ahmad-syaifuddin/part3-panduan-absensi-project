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

Fitur ini sangat penting agar sistem absensi Anda tidak salah menandai karyawan sebagai "Alpa" pada hari libur nasional atau cuti bersama yang ditentukan perusahaan.

Kita akan membangun modul CRUD (Create, Read, Update, Delete) penuh untuk ini.

## Tahap 11: Manajemen Hari Libur (CRUD)
🎯 Tujuan:
Membuat halaman admin untuk menambah, melihat, mengedit, dan menghapus data hari libur. Kemudian, mengintegrasikan data ini ke dalam logika absensi.

Langkah 1: Membuat Model
Database dan migrasi untuk holidays sudah Anda miliki dari file .sql. Sekarang, kita hanya perlu membuat Model-nya agar Laravel bisa berinteraksi dengan tabel tersebut.

Jalankan perintah ini di terminal:

```Bash
php artisan make:model Holiday
```
Buka file yang baru dibuat di app/Models/Holiday.php dan tambahkan properti fillable. Ini menentukan kolom mana yang boleh diisi secara massal.

```PHP    
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Factories\HasFactory;
use Illuminate\Database\Eloquent\Model;

class Holiday extends Model
{
    use HasFactory;

    /**
     * The attributes that are mass assignable.
     *
     * @var array<int, string>
     */
    protected $fillable = [
        'date',
        'description',
    ];
}
```
Langkah 2: Membuat Controller Resourceful
Untuk menangani semua aksi CRUD, kita akan menggunakan Resource Controller. Ini akan secara otomatis membuatkan kita method-method yang dibutuhkan (index, create, store, edit, update, destroy).

Jalankan perintah ini:

```Bash
php artisan make:controller Admin/HolidayController --resource
```
Sekarang, buka app/Http/Controllers/Admin/HolidayController.php dan isi semua logikanya:

```PHP
<?php

namespace App\Http\Controllers\Admin;

use App\Http\Controllers\Controller;
use App\Models\Holiday;
use Illuminate\Http\Request;

class HolidayController extends Controller
{
    public function index()
    {
        $holidays = Holiday::latest()->paginate(10);
        return view('admin.holidays.index', compact('holidays'));
    }

    public function create()
    {
        return view('admin.holidays.create');
    }

    public function store(Request $request)
    {
        $request->validate([
            'date' => 'required|date|unique:holidays,date',
            'description' => 'required|string|max:255',
        ]);

        Holiday::create($request->all());

        return redirect()->route('holidays.index')
                         ->with('success', 'Hari libur berhasil ditambahkan.');
    }

    public function edit(Holiday $holiday)
    {
        return view('admin.holidays.edit', compact('holiday'));
    }

    public function update(Request $request, Holiday $holiday)
    {
        $request->validate([
            'date' => 'required|date|unique:holidays,date,' . $holiday->id,
            'description' => 'required|string|max:255',
        ]);

        $holiday->update($request->all());

        return redirect()->route('holidays.index')
                         ->with('success', 'Hari libur berhasil diperbarui.');
    }

    public function destroy(Holiday $holiday)
    {
        $holiday->delete();

        return redirect()->route('holidays.index')
                         ->with('success', 'Hari libur berhasil dihapus.');
    }
}
```
Langkah 3: Membuat Rute Resourceful
Karena kita menggunakan Resource Controller, kita juga bisa mendefinisikan semua rute CRUD hanya dengan satu baris kode.

Buka routes/web.php dan tambahkan rute ini di dalam grup middleware admin Anda:

```PHP
// routes/web.php
use App\Http\Controllers\Admin\HolidayController;

Route::middleware(['auth'])->group(function () { 
    // ... rute admin lainnya ...
    
    // RUTE BARU UNTUK CRUD HARI LIBUR
    Route::resource('/admin/holidays', HolidayController::class)
        ->except(['show']); // Method 'show' tidak kita gunakan
});
```
Langkah 4: Membuat Views (Halaman Tampilan)
Kita butuh 3 file view: untuk menampilkan daftar (index), form tambah (create), dan form edit (edit).

Buat folder baru: resources/views/admin/holidays.

Di dalamnya, buat 3 file berikut:

File 1: index.blade.php (untuk menampilkan daftar)

```Blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Manajemen Hari Libur') }}
        </h2>
    </x-slot>

    <div class="py-12">
        <div class="max-w-7xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 text-gray-900">
                    <div class="flex justify-end mb-4">
                        <a href="{{ route('holidays.create') }}" class="bg-green-500 hover:bg-green-700 text-white font-bold py-2 px-4 rounded">
                            Tambah Hari Libur
                        </a>
                    </div>
                    
                    @if (session('success'))
                        <div class="mb-4 bg-green-100 border border-green-400 text-green-700 px-4 py-3 rounded">
                            {{ session('success') }}
                        </div>
                    @endif

                    <table class="min-w-full divide-y divide-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Tanggal</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Keterangan</th>
                                <th class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase">Aksi</th>
                            </tr>
                        </thead>
                        <tbody class="bg-white divide-y divide-gray-200">
                            @forelse ($holidays as $holiday)
                                <tr>
                                    <td class="px-6 py-4">{{ \Carbon\Carbon::parse($holiday->date)->isoFormat('dddd, D MMMM Y') }}</td>
                                    <td class="px-6 py-4">{{ $holiday->description }}</td>
                                    <td class="px-6 py-4 whitespace-nowrap text-sm font-medium">
                                        <a href="{{ route('holidays.edit', $holiday) }}" class="text-indigo-600 hover:text-indigo-900">Edit</a>
                                        <form action="{{ route('holidays.destroy', $holiday) }}" method="POST" class="inline-block ml-4" onsubmit="return confirm('Apakah Anda yakin ingin menghapus data ini?');">
                                            @csrf
                                            @method('DELETE')
                                            <button type="submit" class="text-red-600 hover:text-red-900">Hapus</button>
                                        </form>
                                    </td>
                                </tr>
                            @empty
                                <tr>
                                    <td colspan="3" class="px-6 py-4 text-center">Data hari libur tidak ditemukan.</td>
                                </tr>
                            @endforelse
                        </tbody>
                    </table>
                    <div class="mt-4">
                        {{ $holidays->links() }}
                    </div>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
File 2: create.blade.php (form untuk menambah data)

```Blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Tambah Hari Libur Baru') }}
        </h2>
    </x-slot>
    <div class="py-12">
        <div class="max-w-2xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 bg-white border-b border-gray-200">
                    <form method="POST" action="{{ route('holidays.store') }}">
                        @csrf
                        <div>
                            <label for="date" class="block font-medium text-sm text-gray-700">Tanggal</label>
                            <input id="date" type="date" name="date" class="block mt-1 w-full rounded-md shadow-sm border-gray-300" required>
                        </div>
                        <div class="mt-4">
                            <label for="description" class="block font-medium text-sm text-gray-700">Keterangan</label>
                            <input id="description" type="text" name="description" class="block mt-1 w-full rounded-md shadow-sm border-gray-300" required>
                        </div>
                        <div class="flex items-center justify-end mt-4">
                            <button type="submit" class="ml-4 bg-indigo-500 text-white font-bold py-2 px-4 rounded">
                                Simpan
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
File 3: edit.blade.php (form untuk mengubah data)

```Blade
<x-app-layout>
    <x-slot name="header">
        <h2 class="font-semibold text-xl text-gray-800 leading-tight">
            {{ __('Edit Hari Libur') }}
        </h2>
    </x-slot>
    <div class="py-12">
        <div class="max-w-2xl mx-auto sm:px-6 lg:px-8">
            <div class="bg-white overflow-hidden shadow-sm sm:rounded-lg">
                <div class="p-6 bg-white border-b border-gray-200">
                    <form method="POST" action="{{ route('holidays.update', $holiday) }}">
                        @csrf
                        @method('PUT')
                        <div>
                            <label for="date" class="block font-medium text-sm text-gray-700">Tanggal</label>
                            <input id="date" type="date" name="date" value="{{ $holiday->date }}" class="block mt-1 w-full rounded-md shadow-sm border-gray-300" required>
                        </div>
                        <div class="mt-4">
                            <label for="description" class="block font-medium text-sm text-gray-700">Keterangan</label>
                            <input id="description" type="text" name="description" value="{{ $holiday->description }}" class="block mt-1 w-full rounded-md shadow-sm border-gray-300" required>
                        </div>
                        <div class="flex items-center justify-end mt-4">
                            <button type="submit" class="ml-4 bg-indigo-500 text-white font-bold py-2 px-4 rounded">
                                Update
                            </button>
                        </div>
                    </form>
                </div>
            </div>
        </div>
    </div>
</x-app-layout>
```
Langkah 5: Menambahkan Link Navigasi
Buka kembali resources/views/layouts/navigation.blade.php dan tambahkan link baru untuk "Hari Libur" di menu admin.

```blade
<x-nav-link :href="route('dashboard')" :active="request()->routeIs('dashboard')">
    {{ __('Dashboard') }}
</x-nav-link>
<x-nav-link :href="route('admin.reports.daily')" :active="request()->routeIs('admin.reports.daily')">
    {{ __('Laporan Harian') }}
</x-nav-link>
{{-- LINK BARU UNTUK HARI LIBUR --}}
<x-nav-link :href="route('holidays.index')" :active="request()->routeIs('holidays.*')">
    {{ __('Hari Libur') }}
</x-nav-link>
```
Langkah 6: Integrasi dengan Logika "Mark Alpa"
Ini langkah terakhir tapi paling krusial. Kita modifikasi fungsi markAlpa agar tidak berjalan jika hari itu adalah hari libur.

Buka app/Http/Controllers/AttendanceController.php dan update method markAlpa.

```PHP
// app/Http/Controllers/AttendanceController.php

use App\Models\Holiday; // <-- Tambahkan ini di atas

class AttendanceController extends Controller
{
    // ...

    public function markAlpa(Request $request)
    {
        // ... (validasi admin) ...

        $timezone = 'Asia/Makassar';
        $today = Carbon::now($timezone)->format('Y-m-d');
        $now = Carbon::now($timezone);

        // --- PENAMBAHAN LOGIKA BARU ---
        // Cek apakah hari ini adalah hari libur yang terdaftar di database
        $isHoliday = Holiday::where('date', $today)->exists();

        if ($isHoliday) {
            return redirect()->route('dashboard')->with('info', 'Tidak ada tindakan yang diambil karena hari ini adalah hari libur.');
        }
        // --- AKHIR PENAMBAHAN LOGIKA ---

        if ($now->isWeekend()) {
            return redirect()->route('dashboard')->with('info', 'Tidak ada tindakan yang diambil pada hari libur.');
        }

        // ... (sisa logika sama seperti sebelumnya) ...
    }
}
```
## ✅ Uji Coba
Login sebagai Admin.

Buka menu "Hari Libur". Halaman akan menampilkan daftar hari libur (masih kosong).

Klik "Tambah Hari Libur", isi tanggal untuk besok dan keterangannya, lalu simpan.

Coba Edit dan Hapus data yang baru Anda buat untuk memastikan semua fungsi berjalan.

Tambahkan data hari libur untuk hari ini.

Kembali ke Dashboard Admin dan coba klik tombol "Proses & Tandai Karyawan Alpa". Seharusnya Anda akan mendapatkan pesan bahwa tidak ada tindakan yang diambil karena hari ini adalah hari libur.

---

### Next lanjut Part 4 untuk [Panduannya](https://github.com/ahmad-syaifuddin/part4-panduan-absensi-project.git).
