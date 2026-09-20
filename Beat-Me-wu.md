# Write up: [RE] Beat Me

## Mô tả đề bài

Một nhà soạn nhạc yêu nước đang viết dở bài hát động viên anh em nhưng bị thế lực thù địch cài cắm mã độc và mã hoá mất lời bài hát. Hãy giúp tác giả giải mã file lời bài hát.

## Thu thập thông tin

Khi tải file về thì ta thấy 2 file là flag.txt.rnwd và chall.exe.
Tìm kiếm trên mạng thì không thấy file .rnwd là loại file gì nên ta có thể đoán đó là file đang viết dở nhưng lại bị mã goá bởi thế lực thù địch.
Ném file chall.exe vào IDA.
Kiểm tra các entry point, ta thấy mỗi hàm start.

<img width="541" height="163" alt="image" src="https://github.com/user-attachments/assets/def45332-bdac-4541-866c-9a6e6daa5345" />

Kiểm tra hàm start, ta thấy hàm start gọi đến 2 hàm khác là `sub_140A49F40` và `sub_140A49730`

```asm

; __int64 start()
public start
start proc near
sub     rsp, 28h
call    sub_140A49F40
add     rsp, 28h
jmp     sub_140A49730
start endp

```

Truy cập vào trong từng hàm để kiểm tra từng chức năng của hàm `sub_140A49F40`:

```

uintptr_t sub_140A49F40()
{
  uintptr_t result; // rax
  uintptr_t v1; // rcx
  struct _FILETIME v2; // [rsp+20h] [rbp-18h] BYREF
  struct _FILETIME SystemTimeAsFileTime; // [rsp+40h] [rbp+8h] BYREF
  LARGE_INTEGER PerformanceCount; // [rsp+48h] [rbp+10h] BYREF

  if ( _security_cookie == 0x2B992DDFA232LL )
  {
    SystemTimeAsFileTime = 0;
    GetSystemTimeAsFileTime(&SystemTimeAsFileTime);
    v2 = SystemTimeAsFileTime;
    v2 = (struct _FILETIME)(GetCurrentThreadId() ^ *(unsigned __int64 *)&v2);
    v2 = (struct _FILETIME)(GetCurrentProcessId() ^ *(unsigned __int64 *)&v2);
    QueryPerformanceCounter(&PerformanceCount);
    v1 = ((unsigned __int64)&v2
        ^ *(_QWORD *)&v2
        ^ PerformanceCount.QuadPart
        ^ ((unsigned __int64)PerformanceCount.LowPart << 32))
       & 0xFFFFFFFFFFFFLL;
    result = 0x2B992DDFA233LL;
    if ( v1 == 0x2B992DDFA232LL )
      v1 = 0x2B992DDFA233LL;
    _security_cookie = v1;
    qword_140ACC1C0 = ~v1;
  }
  else
  {
    qword_140ACC1C0 = ~_security_cookie;
    return ~_security_cookie;
  }
  return result;
}

```

Ta thấy các câu lệnh như `GetCurrentThreadId` và `GetSystemTimeAsFileTime` thì ta đoán đây là hàm kiểm tra các thông số máy.
Ta thay đổi tên của hàm `sub_140A49F40` thành `hamkiemtra`

Tiếp tục kiểm tra hàm còn lại:

```

__int64 sub_140A49730()
{
  __int64 v0; // rcx
  char v1; // si
  unsigned __int8 v2; // bl
  _QWORD *v4; // rax
  void (__fastcall **v5)(_QWORD, __int64, _QWORD); // rbx
  _QWORD *v6; // rax
  _QWORD *v7; // rbx
  __int64 v8; // rdi
  __int64 v9; // rbx
  unsigned int *v10; // rax
  unsigned int v11; // ebx
  __int64 v12; // rcx

  if ( !(unsigned __int8)sub_140A49C30(1) )
    RtlFailFast(7);
  v1 = 0;
  v2 = sub_140A49A20(v0);
  if ( dword_140ACEDE0 == 1 )
    RtlFailFast(7);
  if ( dword_140ACEDE0 )
  {
    v1 = 1;
  }
  else
  {
    dword_140ACEDE0 = 1;
    if ( (unsigned int)sub_140A8E7D8(&unk_140AB9400, &unk_140AB9438) )
      return 255;
    sub_140A8E7A0(&unk_140AB74A8, &unk_140AB93F8);
    dword_140ACEDE0 = 2;
  }
  sub_140A49DB0(v2);
  v4 = (_QWORD *)sub_140A4A0F0();
  v5 = (void (__fastcall **)(_QWORD, __int64, _QWORD))v4;
  if ( *v4 && (unsigned __int8)sub_140A49D10(v4) )
    (*v5)(0, 2, 0);
  v6 = (_QWORD *)sub_140A4A100();
  v7 = v6;
  if ( *v6 && (unsigned __int8)sub_140A49D10(v6) )
    sub_140A8EC7C(*v7);
  v8 = sub_140A8E6F0();
  v9 = *(_QWORD *)sub_140A8EF5C();
  v10 = (unsigned int *)sub_140A8EF54();
  v11 = sub_1400010D0(*v10, v9, v8);
  if ( !(unsigned __int8)sub_140A4A1B0() )
  {
    sub_140A8ECB8(v11);
    sub_140A8EC68(v11);
    JUMPOUT(0x140A498AFLL);
  }
  if ( !v1 )
    sub_140A8EC58();
  LOBYTE(v12) = 1;
  sub_140A49DE0(v12, 0);
  return v11;
}

```

Ta thấy dòng ` v11 = sub_1400010D0(*v10, v9, v8);` được gọi tới với 3 thông số được khởi tạo bên trên với 3 tham số giống như một hàm main chuẩn.
Truy cập thử vào trong hàm `sub_1400010D0`:

```

__int64 sub_1400010D0()
{
  DWORD v1; // [rsp+2Ch] [rbp-10Ch]
  CHAR Dst[260]; // [rsp+30h] [rbp-108h] BYREF
  int v3; // [rsp+134h] [rbp-4h]

  v3 = 0;
  v1 = ExpandEnvironmentStringsA("%USERPROFILE%\\Desktop\\target", Dst, 0x104u);
  if ( v1 && v1 <= 0x104 )
  {
    if ( GetFileAttributesA(Dst) == -1 )
    {
      return 1;
    }
    else
    {
      sub_140036ED0();
      sub_140001170(Dst);
      sub_140001270(Dst);
      return 0;
    }
  }
  else
  {
    return 1;
  }
}

```

Ta thấy một được dẫn tìm file được truyền cho biến `Dst` và ngay bên dưới nó sẽ kiếm tra xem đường dẫn tuyền vào có thư mục chứa cần tìm không, nếu không thì sẽ trả về 1 biểu thị mã lỗi là 1.
Còn không thì sẽ gọi đến 3 hàm khác nhau để thực hiện.
Từ đó ta thấy được đây chính xác là hàm main chính và một trong 3 hàm kia sẽ có hàm thuật toán và hàm mã độc.
Ta sẽ thử kiểm tra với những hàm mà truyền vào biến `Dst`.
Trong hàm `sub_140001170` ta thấy được rằng nó sẽ tìm tất cả file có trong thư mục mà biến `Dst` được truyền vào thông qua vòng lặp:

```

do
    {
      if ( (FindFileData.dwFileAttributes & 0x10) == 0 )
      {
        if ( (unsigned int)sub_140001340(FindFileData.cFileName) )
        {
          sub_140001FD0(v2, 260, "%s\\%s", v6, FindFileData.cFileName);
          sub_1400013E0(v2);
        }
      }
    }
    while ( FindNextFileA(hFindFile, &FindFileData) );

```

Từ đó ta biết được rằng hàm `sub_140001170` sẽ là hàm tìm file trong thư mục và nếu thấy thì nó sẽ trả về tên file, nếu không thì nó sẽ trả về mã lỗi.
Ta đổi tên file thành `timfile`

Tiếp tục kiểm tra hàm `sub_140001270`:

```

__int64 __fastcall sub_140001270(const char *a1)
{
  unsigned int v1; // eax
  _BYTE v3[272]; // [rsp+30h] [rbp-118h] BYREF
  const char *v4; // [rsp+140h] [rbp-8h]

  v4 = a1;
  sub_140001FD0((__int64)v3, 260, "%s\\HOW_TO_DECRYPT.txt", a1);
  v1 = sub_140AB5690(
         "YOUR FILES HAVE BEEN ENCRYPTED BY UNWINDCRYPT\n"
         "=============================================\n"
         "\n"
         "All .txt, .jpg, .docx files in this directory\n"
         "have been encrypted with a modified AES-256 CBC.\n"
         "\n"
         "To recover your files, you must understand the\n"
         "encryption algorithm and implement the decryption.\n"
         "\n");
  return sub_140001860(
           v3,
           "YOUR FILES HAVE BEEN ENCRYPTED BY UNWINDCRYPT\n"
           "=============================================\n"
           "\n"
           "All .txt, .jpg, .docx files in this directory\n"
           "have been encrypted with a modified AES-256 CBC.\n"
           "\n"
           "To recover your files, you must understand the\n"
           "encryption algorithm and implement the decryption.\n"
           "\n",
           v1);
}

```

Ta thấy đượch rằng hàm này cho biết file bị mã hoá theo cách nào và cách để giải mã nó.
Ta đổi tên file này thành `hint`.

Kiểm tra nốt hàm còn lại thì thấy hàm đó gọi đến hàm khác là `sub_140036EF0`.
Truy cập vào hàm được gọi đến:

```

void __fastcall sub_140036EF0(__int64 a1, __int64 a2)
{
....
  for ( i = 0; i < 8; ++i )
  {
    v31 = (unsigned int)(4 * i);
    v30 = 4 * i;
    dwErrCode = GetLastError();
    v28 = v31 ^ v30 ^ 0x959F4DB3;
    hFindFile = FindFirstFileW("C", &FindFileData);
    if ( hFindFile != (HANDLE)-1LL )
    {
      GetTempPathW(0x104u, Buffer);
      GetTempFileNameW(Buffer, L"aux", 0, TempFileName);
      DeleteFileW(TempFileName);
      v2 = lstrlenW(TempFileName);
      v28 ^= -1640531527 * v2;
      ppsmemCounters.cb = 72;
      CurrentProcess = GetCurrentProcess();
      if ( K32GetProcessMemoryInfo(CurrentProcess, &ppsmemCounters, 0x48u) )
      {
        v28 += LODWORD(ppsmemCounters.WorkingSetSize);
        v28 = LODWORD(ppsmemCounters.PagefileUsage) ^ (1000003 * v28);
      }
      QueryPerformanceFrequency(&Frequency);
      QueryPerformanceCounter(&PerformanceCount);
      v28 = Frequency.LowPart ^ PerformanceCount.LowPart ^ (32 * v28);
      hSCManager = OpenSCManagerW(nullptr, nullptr, 4u);
      if ( hSCManager )
      {
        pcbBytesNeeded = 0;
        ServicesReturned = 0;
        ResumeHandle = 0;
        EnumServicesStatusW(
          hSCManager,
          0x30u,
          3u,
          &Services,
          0x1000u,
          &pcbBytesNeeded,
          &ServicesReturned,
          &ResumeHandle);
        if ( ServicesReturned )
        {
          p_Services = &Services;
          for ( j = 0; ; ++j )
          {
            v6 = 0;
            if ( j < ServicesReturned )
              v6 = j < 8;
            if ( !v6 )
              break;
            v28 = p_Services[j].ServiceStatus.dwCurrentState ^ (32 * v28);
          }
        }
        CloseServiceHandle(hSCManager);
      }
      v12 = ExpandEnvironmentStringsW(L"%LOCALAPPDATA%", Dst, 0x104u);
      v28 = v12 ^ (32 * v28);
      FindClose(hFindFile);
    }
    SetLastError(dwErrCode);
    v5 = (*(unsigned __int8 *)(a1 + 4 * i + 2) << 8)
       | (*(unsigned __int8 *)(4 * i + 1 + a1) << 16)
       | (*(unsigned __int8 *)(4 * i + a1) << 24);
    v50 = 4 * i + 1 + a1;
    v49 = a1;
    LastError = GetLastError();
    v47 = v50 ^ v49 ^ 0xAC58BC7D;
    phkResult = nullptr;
    if ( !RegOpenKeyExW(HKEY_LOCAL_MACHINE, L"SOFTWARE\\ObfJunk\\XYZ", 0, 0x20019u, &phkResult) )
    {
      GetWindowsDirectoryW(String2, 0x104u);
      lstrcpyW(String1, String2);
      lstrcatW(String1, "\\");
      FirstFileW = FindFirstFileW(String1, &v43);
      if ( FirstFileW != (HANDLE)-1LL )
      {
        v41 = 0;
        do
        {
          v47 ^= v43.nFileSizeHigh ^ v43.nFileSizeLow;
          ++v41;
          NextFileW = 0;
          if ( v41 < 12 )
            NextFileW = FindNextFileW(FirstFileW, &v43);
        }
        while ( NextFileW );
        FindClose(FirstFileW);
      }
      hKey = nullptr;
      if ( !RegOpenKeyExW(HKEY_LOCAL_MACHINE, &word_140ABD7BE, 0, 1u, &hKey) )
      {
        cbData = 256;
        RegQueryValueExW(hKey, "E", nullptr, nullptr, Data, &cbData);
        v47 = cbData ^ (32 * v47);
        RegCloseKey(hKey);
      }
      GetNativeSystemInfo(&SystemInfo);
      v47 = SystemInfo.dwAllocationGranularity ^ (1000003 * v47);
      v47 = SystemInfo.dwNumberOfProcessors ^ (1000003 * v47);
      phProv = 0;
      if ( CryptAcquireContextA(&phProv, nullptr, "Microsoft Enhanced Cryptographic Provider v1.0", 1u, 0xF0000000) )
      {
        *(_DWORD *)pbData = 0;
        pdwDataLen = 4;
        CryptGetProvParam(phProv, 8u, pbData, &pdwDataLen, 1u);
        v47 ^= *(_DWORD *)pbData;
        CryptReleaseContext(phProv, 0);
      }
      v32 = ExpandEnvironmentStringsW("%", v33, 0x104u);
      v47 = v32 ^ (1000003 * v47);
      RegCloseKey(phkResult);
    }
    SetLastError(LastError);
    *(_DWORD *)(a2 + 4LL * i) = *(unsigned __int8 *)(a1 + 4 * i + 3) | v5;
  }
  for ( k = 8; k < 60; ++k )
  {
    v7 = *(_DWORD *)(a2 + 4LL * (k - 1));
    if ( k % 8 )
    {
      if ( k % 8 == 4 )
        v7 = sub_14003ABA0(v7);
    }
    else
    {
      v7 = dword_140ABD510[k / 8 - 1] ^ sub_14003ABA0(HIBYTE(v7) | (v7 << 8));
    }
    *(_DWORD *)(a2 + 4LL * k) = v7 ^ *(_DWORD *)(a2 + 4LL * (k - 8));
  }
}

```

Qua đoạn code trên, từ lệnh `RegOpenKeyExW` thì ta biết được đây là lệnh lấy key có sẵn trên hệ thống.
Do đó để tìm được key thì phải debug bài này ở đoạn chính này. 
Nhưng do đây có thể là mã độc nên ta sẽ debug thử trong máy ảo.
Đặt break point ở hàm `start`, 
