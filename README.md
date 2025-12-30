# 🌊 FFT Visualizer (Fast Fourier Transform)

![C++](https://img.shields.io/badge/Language-C++-00599C?style=for-the-badge&logo=c%2B%2B)
![Algorithm](https://img.shields.io/badge/Algorithm-Cooley--Tukey-orange?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-green?style=for-the-badge)

Dự án này là một triển khai thuật toán **Biến đổi Fourier nhanh (Fast Fourier Transform - FFT)** bằng ngôn ngữ C++. Chương trình sử dụng phương pháp **Cooley-Tukey** (Iterative) kết hợp với kỹ thuật **Bit-reversal permutation** để đạt hiệu suất tối ưu $O(N \log N)$.

---

## 🚀 Tính năng nổi bật

* **Xử lý số phức:** Sử dụng thư viện `<complex>` chuẩn của C++ để tính toán chính xác phần thực và phần ảo.
* **Tối ưu hóa:** Sử dụng kỹ thuật đảo bit (Bit-reversal) để sắp xếp lại mảng đầu vào, cho phép tính toán tại chỗ (in-place computation).
* **Tự động Padding:** Tự động chèn thêm số 0 nếu kích thước đầu vào không phải là lũy thừa của 2.
* **Hỗ trợ FFT ngược:** Code được thiết kế để dễ dàng mở rộng sang Inverse FFT (IFFT).

---

## 🛠️ Cài đặt và Chạy chương trình

Bạn chỉ cần một trình biên dịch C++ (như G++).

### 1. Biên dịch
Mở terminal tại thư mục dự án và chạy lệnh:

```bash
g++ main.cpp -o fft
```

### 2. Chạy chương trình
```bash
# Trên Windows
./fft.exe

# Trên Linux/Mac
./fft
```

---

## 🧠 Giải thích thuật toán

Mã nguồn bao gồm 2 thành phần chính:

### 1. Bit Reversal (Đảo Bit)
Trước khi thực hiện FFT, mảng đầu vào được sắp xếp lại theo thứ tự đảo ngược bit của chỉ số. Điều này giúp thuật toán Cooley-Tukey có thể thực hiện theo kiểu lặp (iterative) thay vì đệ quy.

### 2. Butterfly Operation (Cánh bướm)
Thuật toán sử dụng công thức:
$$ a[k] = u + w \cdot v $$
$$ a[k + n/2] = u - w \cdot v $$

---

## 💻 Mã nguồn (Source Code)

```cpp
#include <iostream>
#include <vector>
#include <complex>
#include <cmath>
#include <algorithm>
#include <iomanip>

using namespace std;

// Dùng số phức để tính toán nhanh hơn
using cd = complex<double>;
const double PI = acos(-1); // Số PI

// Hàm đảo vị trí theo bit - chuẩn bị cho FFT
void bit_reverse(vector<cd> & a) {
    int n = a.size();
    for (int i = 1, j = 0; i < n; i++) {
        int bit = n >> 1;
        while (j & bit) {
            j ^= bit;
            bit >>= 1;
        }
        j ^= bit;
        if (i < j) swap(a[i], a[j]);
    }
}

// Thuật toán FFT chính
void fft(vector<cd> & a, bool invert) {
    int n = a.size();

    // Bước 1: Đảo bit trước
    bit_reverse(a);

    // Bước 2: Tính toán từng tầng
    for (int len = 2; len <= n; len <<= 1) {
        double angle = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(angle), sin(angle));

        for (int i = 0; i < n; i += len) {
            cd w(1);
            for (int j = 0; j < len / 2; j++) {
                cd u = a[i + j];
                cd v = a[i + j + len / 2] * w;

                a[i + j] = u + v;
                a[i + j + len / 2] = u - v;

                w *= wlen;
            }
        }
    }

    if (invert) {
        for (cd & x : a) x /= n;
    }
}

int main() {
    cout << "Nhập số lượng phần tử (nên là lũy thừa của 2, ví dụ: 4, 8): ";
    int n_input;
    cin >> n_input;

    vector<cd> a;
    cout << "Nhập các giá trị thực: ";
    for(int i = 0; i < n_input; ++i) {
        double val;
        cin >> val;
        a.push_back(cd(val, 0));
    }

    // Padding (thêm số 0)
    int n = 1;
    while (n < a.size()) n <<= 1;
    a.resize(n);

    cout << "\n--- Bắt đầu tính FFT (Kích thước N = " << n << ") ---\n";
    fft(a, false);

    cout << fixed << setprecision(2);
    cout << "Kết quả sau khi biến đổi:\n";
    for (int i = 0; i < n; i++) {
        cout << "Vị trí " << i << ": " << a[i].real();
        if (a[i].imag() >= 0) cout << " + " << a[i].imag() << "i";
        else cout << " - " << abs(a[i].imag()) << "i";
        cout << endl;
    }

    return 0;
}
```

---

## 📊 Ví dụ Minh họa

**Input:**
```text
Số lượng phần tử: 4
Giá trị: 1 2 3 4
```

**Output:**
```text
Vị trí 0: 10.00 + 0.00i
Vị trí 1: -2.00 + 2.00i
Vị trí 2: -2.00 + 0.00i
Vị trí 3: -2.00 - 2.00i
```
