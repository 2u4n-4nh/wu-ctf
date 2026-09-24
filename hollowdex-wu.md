# Write up: [RE] Hollowdex

## Thông tin đề bài

### Đề bài

<img width="930" height="442" alt="image" src="https://github.com/user-attachments/assets/6d944de9-7732-42e6-87e5-f13eb4ad99a8" />

### Mô tả đề bài

HollowDex Vault là một ứng dụng Android dùng để kiểm tra thông tin xác thực. Khi nhập đúng khóa hex dài 32 ký tự, ứng dụng sẽ cho phép truy cập.
Tệp APK này có cấu trúc khá đơn giản. Thư viện native được nạp một cách âm thầm. Phương thức Java thực hiện việc kiểm tra nằm ngay trong tệp DEX.
Nhưng thực tế lại không phải vậy.

## Thu thập thông tin

Ném file hollowdex.apk vào trong jadx, kiểm tra hàm `MainAcivity`:

```java script

...
    public /* synthetic */ void lambda$onCreate$0(EditText editText, TextView textView, View view) {
        if (this.verifier.verifyFlag(editText.getText().toString().trim())) {
            textView.setText("ACCESS GRANTED");
        } else {
            textView.setText("ACCESS DENIED");

```

Ta thấy hàm này chỉ để nhập và kiểm tra chuỗi nhập thông qua hàm `verifyFlag` và trả về `"ACCESS GRANTED"` khi đúng và `"ACCESS DENIED"` khi sai

Truy cập vào hàm `verifyFlag`:

```

package com.hollowdex;

/* JADX INFO: loaded from: classes.dex */
public class Verifier {
    /*  JADX ERROR: Dependency scan failed
        jadx.core.utils.exceptions.JadxRuntimeException: Usage info collection failed with error: newPosition > limit: (14820 > 3828) at insn: 0x0007: INVOKE_CUSTOM 
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.lambda$processInstructions$0(UsageInfoVisitor.java:133)
        	at jadx.plugins.input.dex.sections.DexCodeReader.visitInstructions(DexCodeReader.java:86)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.processInstructions(UsageInfoVisitor.java:128)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.processMethod(UsageInfoVisitor.java:113)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.processClass(UsageInfoVisitor.java:101)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.buildUsageData(UsageInfoVisitor.java:81)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.init(UsageInfoVisitor.java:65)
        	at jadx.core.dex.nodes.RootNode.runPreDecompileStage(RootNode.java:376)
        Caused by: java.lang.IllegalArgumentException: newPosition > limit: (14820 > 3828)
        	at java.base/java.nio.Buffer.createPositionException(Unknown Source)
        	at java.base/java.nio.Buffer.position(Unknown Source)
        	at java.base/java.nio.ByteBuffer.position(Unknown Source)
        	at jadx.plugins.input.dex.sections.SectionReader.absPos(SectionReader.java:82)
        	at jadx.plugins.input.dex.sections.SectionReader.getCallSite(SectionReader.java:207)
        	at jadx.plugins.input.dex.insns.DexInsnData.getIndexAsCallSite(DexInsnData.java:141)
        	at jadx.core.utils.input.InsnDataUtils.getCallSite(InsnDataUtils.java:27)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.processInsn(UsageInfoVisitor.java:178)
        	at jadx.core.dex.visitors.usage.UsageInfoVisitor.lambda$processInstructions$0(UsageInfoVisitor.java:130)
        	... 7 more
        */
    public boolean verifyFlag(java.lang.String r14) {
        /*
        // Can't load method instructions: Load method exception: JadxRuntimeException: Failed to decode insn: 0x0006: UNKNOWN(0xC0E9) in method: com.hollowdex.Verifier.verifyFlag(java.lang.String):boolean, file: classes.dex
        */
        throw new UnsupportedOperationException("Method not decompiled: com.hollowdex.Verifier.verifyFlag(java.lang.String):boolean");
    }

    static {
        System.loadLibrary("hollowdex");
    }
}

```

Ta không thể đọc được file này do file đã chặn tool khiến tool không thể đọc được file dex.

Ném file này vào trong máy ảo kali.
Dùng lệnh `unzip -l hollowdex.apk` để giải nén file này.
Ta thấy có file `classes.dex` ở trong, để đọc file đó thì ta dụng lệnh `dexdump -d classes.dex | grep -A 20 "verifyFlag"`:

```

09-24 02:58:50.087 29708 29708 W dexdump : dexdump.cc:1256 GLITCH: run-away instruction at idx=0x9d
000558: 6e20 1200 1300                         |000e: invoke-virtual {v3, v1}, Lcom/hollowdex/Verifier;.verifyFlag:(Ljava/lang/String;)Z // method@0012
00055e: 0a01                                   |0011: move-result v1
000560: 3801 0800                              |0012: if-eqz v1, 001a // +0008
000564: 1a01 0400                              |0014: const-string v1, "ACCESS GRANTED" // string@0004
000568: 6e20 0400 1200                         |0016: invoke-virtual {v2, v1}, Landroid/widget/TextView;.setText:(Ljava/lang/CharSequence;)V // method@0004
00056e: 2806                                   |0019: goto 001f // +0006
000570: 1a01 0300                              |001a: const-string v1, "ACCESS DENIED" // string@0003
000574: 6e20 0400 1200                         |001c: invoke-virtual {v2, v1}, Landroid/widget/TextView;.setText:(Ljava/lang/CharSequence;)V // method@0004
00057a: 0e00                                   |001f: return-void
      catches       : (none)
      positions     :
        0x0000 line=23
        0x000c line=24
        0x0014 line=25
        0x001a line=27
      locals        :
        0x0000 - 0x0020 reg=0 this Lcom/hollowdex/MainActivity;
        0x0000 - 0x0020 reg=1 (null) Landroid/widget/EditText;
        0x0000 - 0x0020 reg=2 (null) Landroid/widget/TextView;
        0x0000 - 0x0020 reg=3 (null) Landroid/view/View;

--
      name          : 'verifyFlag'
      type          : '(Ljava/lang/String;)Z'
      access        : 0x0001 (PUBLIC)
      code          -
      registers     : 15
      ins           : 2
      outs          : 2
      insns size    : 156 16-bit code units
00062c:                                        |[00062c] com.hollowdex.Verifier.verifyFlag:(Ljava/lang/String;)Z
00063c: 4b0e 426b                              |0000: aput v14, v66, v107
000640: 8ec3                                   |0002: int-to-char v3, v12
000642: d0b1 f4c1                              |0003: add-int/lit16 v1, v11, #int -15884 // #c1f4
000646: e981                                   |0005: unused-e9
000648: e9c0                                   |0006: unused-e9
00064a: fc03 790e 4844                         |0007: invoke-custom {}, call_site@0e79
000650: 10c3                                   |000a: return-wide v195
000652: 97a1 73c1                              |000b: xor-int v161, v115, v193
000656: f480                                   |000d: unused-f4
000658: f3c1                                   |000e: unused-f3
00065a: cc13                                   |000f: sub-double/2addr v3, v1
00065c: 430e                                   |0010: unused-43
00065e: 7b66                                   |0011: neg-int v6, v6
000660: 06d7 8bb2 c7c1                         |0012: move-wide/16 v45707, v49607
000666: 3d84 e0c3                              |0015: if-lez v132, ffffc3f5 // -3c20
00066a: 8121                                   |0017: int-to-long v1, v2
00066c: 4f0e 2465                              |0018: aput-byte v14, v36, v101
000670: 1ec5                                   |001a: monitor-exit v197
000672: cf81                                   |001b: rem-double/2addr v1, v8
000674: f0c1                                   |001c: unused-f0

```

Trong đoạn code trên thì đoạn này nó cũng giống như đoạn code bên trên là cũng bị chặn bởi tác giả nên ta chuyển sang xem thử file `libhollowdex.so`.
Ném file này vào ghidra và tìm hàm `JNI_OnLoad` :

```


undefined4 JNI_OnLoad(long *param_1)

{
  char *__offset;
  uint uVar1;
  int iVar2;
  long lVar3;
  FILE *__stream;
  char *pcVar4;
  void *__buf;
  ulong uVar5;
  ulong uVar6;
  undefined8 *puVar7;
  ulong __n;
  undefined8 uVar8;
  undefined8 uVar9;
  undefined8 uVar10;
  undefined8 uVar11;
  long *local_270;
  undefined1 auStack_268 [8];
  char local_260 [16];
  undefined8 uStack_250;
  undefined8 uStack_248;
  char *local_48;
  
  local_270 = (long *)0x0;
  iVar2 = (**(code **)(*param_1 + 0x30))(param_1,&local_270,0x10006);
  if (((iVar2 != 0) ||
      (lVar3 = (**(code **)(*local_270 + 0x30))(local_270,"com/hollowdex/Verifier"), lVar3 == 0)) ||
     (lVar3 = (**(code **)(*local_270 + 0x108))
                        (local_270,lVar3,"verifyFlag","(Ljava/lang/String;)Z"), lVar3 == 0)) {
    return 0xffffffff;
  }
  uVar1 = *(uint *)(lVar3 + 8);
  __stream = fopen("/proc/self/maps","r");
  if (__stream == (FILE *)0x0) {
    return 0x10006;
  }
  pcVar4 = fgets(local_260,0x200,__stream);
  while (pcVar4 != (char *)0x0) {
    pcVar4 = strstr(local_260,".apk");
    if ((((pcVar4 != (char *)0x0) &&
         ((pcVar4 = strstr(local_260,"r--p"), pcVar4 != (char *)0x0 ||
          (pcVar4 = strstr(local_260,"r-xp"), pcVar4 != (char *)0x0)))) &&
        (iVar2 = sscanf(local_260,"%lx-%lx",&local_48,auStack_268), iVar2 == 2)) &&
       (((*local_48 == 'd' && (local_48[1] == 'e')) &&
        ((local_48[2] == 'x' && (pcVar4 = local_48, local_48[3] == '\n')))))) goto LAB_00101aa8;
    pcVar4 = fgets(local_260,0x200,__stream);
  }
  pcVar4 = (char *)0x0;
LAB_00101aa8:
  fclose(__stream);
  if (pcVar4 == (char *)0x0) {
    return 0x10006;
  }
  if (uVar1 == 0) {
    return 0x10006;
  }
  pcVar4 = pcVar4 + uVar1;
  __offset = pcVar4 + 0x10;
  uVar1 = *(int *)(pcVar4 + 0xc) << 1;
  __n = (ulong)uVar1;
  local_260[8] = -0x1d;
  local_260[9] = -0x3f;
  local_260[10] = -0x19;
  local_260[0xb] = -0x7f;
  local_260[0xc] = -0x1d;
  local_260[0xd] = -0x3f;
  local_260[0xe] = -0x11;
  local_260[0xf] = '\x01';
  local_260[0] = 'Y';
  local_260[1] = '\x0e';
  local_260[2] = 'z';
  local_260[3] = 'e';
  local_260[4] = '\x14';
  local_260[5] = -0x3d;
  local_260[6] = -0x42;
  local_260[7] = -0x5f;
  __buf = operator.new[](__n);
  if (uVar1 != 0) {
    uVar5 = 0;
    if (((7 < uVar1) && (0xffffffffffffffef < __n - 0x11)) &&
       (0x1f < (ulong)((long)__buf - (long)__offset))) {
      if (0x1f < uVar1) {
        puVar7 = (undefined8 *)((long)__buf + 0x10);
        do {
          uVar9 = *(undefined8 *)(pcVar4 + 0x18);
          uVar8 = *(undefined8 *)(pcVar4 + 0x10);
          uVar11 = *(undefined8 *)(pcVar4 + 0x28);
          uVar10 = *(undefined8 *)(pcVar4 + 0x20);
          puVar7[-1] = CONCAT17(SUB81(local_260._8_8_,7) ^ (byte)((ulong)uVar9 >> 0x38),
                                CONCAT16(SUB81(local_260._8_8_,6) ^ (byte)((ulong)uVar9 >> 0x30),
                                         CONCAT15(SUB81(local_260._8_8_,5) ^
                                                  (byte)((ulong)uVar9 >> 0x28),
                                                  CONCAT14(SUB81(local_260._8_8_,4) ^
                                                           (byte)((ulong)uVar9 >> 0x20),
                                                           CONCAT13(SUB81(local_260._8_8_,3) ^
                                                                    (byte)((ulong)uVar9 >> 0x18),
                                                                    CONCAT12(SUB81(local_260._8_8_,2
                                                                                  ) ^ (byte)((ulong)
                                                  uVar9 >> 0x10),
                                                  CONCAT11(SUB81(local_260._8_8_,1) ^
                                                           (byte)((ulong)uVar9 >> 8),
                                                           (byte)local_260._8_8_ ^ (byte)uVar9))))))
                               );
          puVar7[-2] = CONCAT17(SUB81(local_260._0_8_,7) ^ (byte)((ulong)uVar8 >> 0x38),
                                CONCAT16(SUB81(local_260._0_8_,6) ^ (byte)((ulong)uVar8 >> 0x30),
                                         CONCAT15(SUB81(local_260._0_8_,5) ^
                                                  (byte)((ulong)uVar8 >> 0x28),
                                                  CONCAT14(SUB81(local_260._0_8_,4) ^
                                                           (byte)((ulong)uVar8 >> 0x20),
                                                           CONCAT13(SUB81(local_260._0_8_,3) ^
                                                                    (byte)((ulong)uVar8 >> 0x18),
                                                                    CONCAT12(SUB81(local_260._0_8_,2
                                                                                  ) ^ (byte)((ulong)
                                                  uVar8 >> 0x10),
                                                  CONCAT11(SUB81(local_260._0_8_,1) ^
                                                           (byte)((ulong)uVar8 >> 8),
                                                           (byte)local_260._0_8_ ^ (byte)uVar8))))))
                               );
          puVar7[1] = CONCAT17((byte)((ulong)uStack_248 >> 0x38) ^ (byte)((ulong)uVar11 >> 0x38),
                               CONCAT16((byte)((ulong)uStack_248 >> 0x30) ^
                                        (byte)((ulong)uVar11 >> 0x30),
                                        CONCAT15((byte)((ulong)uStack_248 >> 0x28) ^
                                                 (byte)((ulong)uVar11 >> 0x28),
                                                 CONCAT14((byte)((ulong)uStack_248 >> 0x20) ^
                                                          (byte)((ulong)uVar11 >> 0x20),
                                                          CONCAT13((byte)((ulong)uStack_248 >> 0x18)
                                                                   ^ (byte)((ulong)uVar11 >> 0x18),
                                                                   CONCAT12((byte)((ulong)uStack_248
                                                                                  >> 0x10) ^
                                                                            (byte)((ulong)uVar11 >>
                                                                                  0x10),
                                                                            CONCAT11((byte)((ulong)
                                                  uStack_248 >> 8) ^ (byte)((ulong)uVar11 >> 8),
                                                  (byte)uStack_248 ^ (byte)uVar11)))))));
          *puVar7 = CONCAT17((byte)((ulong)uStack_250 >> 0x38) ^ (byte)((ulong)uVar10 >> 0x38),
                             CONCAT16((byte)((ulong)uStack_250 >> 0x30) ^
                                      (byte)((ulong)uVar10 >> 0x30),
                                      CONCAT15((byte)((ulong)uStack_250 >> 0x28) ^
                                               (byte)((ulong)uVar10 >> 0x28),
                                               CONCAT14((byte)((ulong)uStack_250 >> 0x20) ^
                                                        (byte)((ulong)uVar10 >> 0x20),
                                                        CONCAT13((byte)((ulong)uStack_250 >> 0x18) ^
                                                                 (byte)((ulong)uVar10 >> 0x18),
                                                                 CONCAT12((byte)((ulong)uStack_250
                                                                                >> 0x10) ^
                                                                          (byte)((ulong)uVar10 >>
                                                                                0x10),
                                                                          CONCAT11((byte)((ulong)
                                                  uStack_250 >> 8) ^ (byte)((ulong)uVar10 >> 8),
                                                  (byte)uStack_250 ^ (byte)uVar10)))))));
          puVar7 = puVar7 + 4;
          pcVar4 = pcVar4 + 0x20;
        } while( true );
      }
      uVar6 = 0;
      uVar5 = __n & 0x18;
      do {
        uVar8 = *(undefined8 *)(__offset + uVar6);
        uVar9 = *(undefined8 *)((ulong)local_260 | uVar6 & 8);
        *(ulong *)((long)__buf + uVar6) =
             CONCAT17((byte)((ulong)uVar9 >> 0x38) ^ (byte)((ulong)uVar8 >> 0x38),
                      CONCAT16((byte)((ulong)uVar9 >> 0x30) ^ (byte)((ulong)uVar8 >> 0x30),
                               CONCAT15((byte)((ulong)uVar9 >> 0x28) ^ (byte)((ulong)uVar8 >> 0x28),
                                        CONCAT14((byte)((ulong)uVar9 >> 0x20) ^
                                                 (byte)((ulong)uVar8 >> 0x20),
                                                 CONCAT13((byte)((ulong)uVar9 >> 0x18) ^
                                                          (byte)((ulong)uVar8 >> 0x18),
                                                          CONCAT12((byte)((ulong)uVar9 >> 0x10) ^
                                                                   (byte)((ulong)uVar8 >> 0x10),
                                                                   CONCAT11((byte)((ulong)uVar9 >> 8
                                                                                  ) ^ (byte)((ulong)
                                                  uVar8 >> 8),(byte)uVar9 ^ (byte)uVar8)))))));
        uVar6 = uVar6 + 8;
      } while (uVar5 != uVar6);
      if (uVar5 == __n) goto LAB_00101b6c;
    }
    do {
      *(byte *)((long)__buf + uVar5) = *(byte *)((ulong)local_260 | uVar5 & 0xf) ^ __offset[uVar5];
      uVar5 = uVar5 + 1;
    } while (__n != uVar5);
  }
LAB_00101b6c:
  iVar2 = open("/proc/self/mem",2);
  if (-1 < iVar2) {
    lseek(iVar2,(__off_t)__offset,0);
    write(iVar2,__buf,__n);
    close(iVar2);
  }
  operator.delete[](__buf);
  return 0x10006;
}

```

Trong đoạn code này ta thấy nó có tìm và khởi tạo hàm `verifyFlag`, đồng thời trng đoạn code trên ta cũng thấy các gía trị cố định:

```C

  local_260[8] = -0x1d;
  local_260[9] = -0x3f;
  local_260[10] = -0x19;
  local_260[0xb] = -0x7f;
  local_260[0xc] = -0x1d;
  local_260[0xd] = -0x3f;
  local_260[0xe] = -0x11;
  local_260[0xf] = '\x01';
  local_260[0] = 'Y';
  local_260[1] = '\x0e';
  local_260[2] = 'z';
  local_260[3] = 'e';
  local_260[4] = '\x14';
  local_260[5] = -0x3d;
  local_260[6] = -0x42;
  local_260[7] = -0x5f

```

Sử dụng
