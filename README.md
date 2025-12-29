# fft-visualizer

# Dưới đây là code C++ mô phỏng thuật toán:
#include <iostream>
#include <vector>
#include <complex>
#include <cmath>
#include <algorithm>
#include <iomanip>

using namespace std;

// Dùng số phức để tính toán nhanh hơn
using cd = complex<double>;
const double PI = acos(-1); // Số PI, dùng acos(-1) cho chính xác

// Hàm đảo vị trí theo bit - chuẩn bị cho FFT
// Cái này giống như xáo bài, sắp xếp lại mảng để tính toán dễ hơn
void bit_reverse(vector<cd> & a) {
    int n = a.size();
    for (int i = 1, j = 0; i < n; i++) {
        int bit = n >> 1;
        // Tìm bit 0 đầu tiên để đảo
        while (j & bit) {
            j ^= bit;
            bit >>= 1;
        }
        j ^= bit;

        // Đổi chỗ nếu cần, tránh đổi 2 lần
        if (i < j)
            swap(a[i], a[j]);
    }
}

// Thuật toán FFT chính
// invert = false: tính FFT bình thường
// invert = true: tính FFT ngược (IFFT)
void fft(vector<cd> & a, bool invert) {
    int n = a.size();

    // Bước 1: Đảo bit trước
    bit_reverse(a);

    // Bước 2: Tính toán từng tầng
    for (int len = 2; len <= n; len <<= 1) {
        // Góc xoay cho mỗi bước
        double angle = 2 * PI / len * (invert ? -1 : 1);
        cd wlen(cos(angle), sin(angle));  // Hệ số xoay

        for (int i = 0; i < n; i += len) {
            cd w(1);
            for (int j = 0; j < len / 2; j++) {
                // Lấy 2 phần tử cần tính
                cd u = a[i + j];
                cd v = a[i + j + len / 2] * w;

                // Công thức butterfly - như cánh bướm
                a[i + j] = u + v;
                a[i + j + len / 2] = u - v;

                // Nhân thêm hệ số xoay
                w *= wlen;
            }
        }
    }

    // Nếu là FFT ngược thì chia cho n để chuẩn hóa
    if (invert) {
        for (cd & x : a)
            x /= n;
    }
}

int main() {
    // Phần nhập liệu
    cout << "Nhập số lượng phần tử (nên là lũy thừa của 2, ví dụ: 4, 8): ";
    int n_input;
    cin >> n_input;

    vector<cd> a;
    cout << "Nhập các giá trị thực: ";
    for(int i = 0; i < n_input; ++i) {
        double val;
        cin >> val;
        a.push_back(cd(val, 0)); // Phần ảo = 0 vì chỉ có số thực
    }

    // Thêm số 0 vào cuối nếu cần (padding)
    int n = 1;
    while (n < a.size()) n <<= 1;
    a.resize(n);

    cout << "\n--- Bắt đầu tính FFT (Kích thước N = " << n << ") ---\n";

    // Gọi hàm FFT
    fft(a, false);

    // In kết quả
    cout << fixed << setprecision(2);
    cout << "Kết quả sau khi biến đổi:\n";
    for (int i = 0; i < n; i++) {
        cout << "Vị trí " << i << ": " << a[i].real();
        if (a[i].imag() >= 0)
            cout << " + " << a[i].imag() << "i";
        else
            cout << " - " << abs(a[i].imag()) << "i";
        cout << endl;
    }

    return 0;
}
