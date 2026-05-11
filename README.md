# 112_Zakharov_KT4
Контрольная точка 4


```cpp
#include <iostream>
#include <string.h>
#include <fstream>
using namespace std;

const int SIZE = 20;

enum MountainType {
    volcanic,
    folded,
    plateau,
    dome,
    other
};

struct Peak {
    char name[80];
    int height;
    char country[80];
    MountainType type;
};

string type_to_string(MountainType t) {
    switch(t) {
        case volcanic: return "volcanic";
        case folded: return "folded";
        case plateau: return "plateau";
        case dome: return "dome";
        default: return "other";
    }
}

// Вывод всех вершин
void print(Peak* a, int size) {
    for (int i = 0; i < size; i++) {
        cout << "Name: " << a[i].name << ", Height: " << a[i].height <<
             ", Country: " << a[i].country << ", Type: " <<
             type_to_string(a[i].type) << endl;
    }
}

bool filter_by_country(Peak a, string country) {
    return a.country == country;
}

void print(Peak* a, int size, string s, bool (*flag) (Peak, string)) {
    for (int i = 0; i < size; i++) {
        if (flag(a[i], s)) {
            cout << "Name: " << a[i].name << ", Height: " << a[i].height <<
                 ", Country: " << a[i].country << ", Type: " <<
                 type_to_string(a[i].type) << endl;
        }
    }
}

bool filter_le_heigt(Peak a, int b) {
    return a.height <= b;
}

void print(Peak* a, int size,int num, bool (*flag) (Peak, int)) {
    for (int i = 0; i < size; i++) {
        if (flag(a[i], num)) {
            cout << "Name: " << a[i].name << ", Height: " << a[i].height <<
                 ", Country: " << a[i].country << ", Type: " <<
                 type_to_string(a[i].type) << endl;
        }
    }
}


// Компаратор: высота вершины a больше высоты вершины b
bool cmp_by_height(Peak a, Peak b) {
    return a.height > b.height;
}


// Сортировка
void sort(Peak* peaks, int size, bool (*cmp) (Peak, Peak)) {
    for (int i = 0; i < size; i++) {
        for (int j = 0; j < size - i - 1; j++) {
            if (cmp(peaks[j] , peaks[j + 1])) {
                Peak temp = peaks[j];
                peaks[j] = peaks[j + 1];
                peaks[j + 1] = temp;
            }
        }
    }
}

// Средняя высота
double average_height(Peak* a, int size) {
    int sum = 0;
    for (int i = 0; i < size; i++) sum += a[i].height;
    return (double)sum / size;
}

// вывод стран нахожнения трех восьмитысячников
void print_counties_8000(Peak* a, int size) {
    cout << "Countries with 8000s:\n";
    int cnt = 0;
    for (int i = 0; i < size && cnt < 3; i++) {
        if (a[i].height >= 8000) {
            cout << a[i].country << '\n';
            cnt++;
        }
    }
}

// Функция изменения данных горной вершины по ее высоте
void change_by_height(Peak* peaks, int size, int height, Peak a) {
    for (int i = 0; i < size; i++) {
        if (peaks[i].height == height) {
            peaks[i] = a;
            return;
        }
    }
    cout << "There's no peak with " << height << " height\n";
}

// Горные вершины одной страны
Peak* peaks_by_country(Peak* a, int size, string country, int& res_size) {
    int cnt = 0;
    for (int i = 0; i < size; i++)
        if (a[i].country == country)
            cnt++;

    res_size = cnt;
    if (cnt == 0) {
        cout << "There are no peaks in that country\n";
        return nullptr;
    }
    Peak* res = new Peak[cnt];
    int pos = 0;
    for (int i = 0; i < size; i++) {
        if (a[i].country == country) {
            res[pos] = a[i];
            pos++;
        }
    }
    return res;
}

// ================================

// Задание 1
void update_heights_from_file(Peak* peaks, int size, const char* filename) {
    ifstream fin(filename);

    char name[80];
    int new_height;

    while (fin >> name >> new_height) {
        for (int i = 0; i < size; i++) {
            if (strcmp(peaks[i].name, name) == 0) {
                peaks[i].height = new_height;
                break;
            }
        }
    }
    fin.close();
}

// Задание 2 запись в бинарный файл
void task2_write(Peak* peaks, int size, const char* filename) {
    ofstream fout(filename, ios::binary);
    fout.write((char*)&size, sizeof(size));
    fout.write((char*)peaks, size * sizeof(Peak));
    fout.close();
}

// Задание 2 чтение из бинарного файла
Peak* task2_read(const char* filename, int& size) {
    ifstream fin(filename, ios::binary);
    if (!fin.is_open()) {
        size = 0;
        return nullptr;
    }
    fin.read((char*)&size, sizeof(size));
    Peak* loaded = new Peak[size];
    fin.read((char*)loaded, size * sizeof(Peak));
    fin.close();
    return loaded;
}

int main() {

    // Ввести информацию по 20 вершинам
    Peak peaks[SIZE] = {
            {"Everest", 8848, "Nepal", folded},
            {"K2", 8611, "Pakistan", folded},
            {"Kangchenjunga", 8586, "Nepal", folded},
            {"Lhotse", 8516, "Nepal", folded},
            {"Makalu", 8485, "Nepal", folded},
            {"Cho Oyu", 8201, "Nepal", folded},
            {"Dhaulagiri", 8167, "Nepal", folded},
            {"Manaslu", 8163, "Nepal", folded},
            {"Nanga Parbat", 8126, "Pakistan", folded},
            {"Annapurna", 8091, "Nepal", folded},
            {"Fuji", 3776, "Japan", volcanic},
            {"Elbrus", 5642, "Russia", volcanic},
            {"Kilimanjaro", 5895, "Tanzania", volcanic},
            {"Mont Blanc", 4808, "France", folded},
            {"Matterhorn", 4478, "Switzerland", folded},
            {"Aconcagua", 6961, "Argentina", folded},
            {"Denali", 6190, "USA", folded},
            {"Cotopaxi", 5897, "Ecuador", volcanic},
            {"Stolovaya", 1085, "Russia", plateau},
            {"Ayu-Dag", 572, "Russia", dome}
    };

    // Вывести среднее значение высот всех 20 вершин
    cout << "Avg height: " << average_height(peaks, SIZE) << "\n\n";

    // Затем вывести информацию, отсортированную по возрастанию высоты
    // вершины (рационально переставлять все поля структуры разом)
    sort(peaks, SIZE, cmp_by_height);
    print(peaks, SIZE);
    cout << "\n";

    // Вывести сведения
    // по странам местонахождения 3-х восьмитысячников
    print_counties_8000(peaks, SIZE);
    cout << "\n";

    // Реализовать функцию
    // изменения данных горной вершины по ее высоте, а не по названию
    Peak new_peak = {"New_Everest", 8848, "Nepal", folded};
    change_by_height(peaks, SIZE, 8848, new_peak);
    print(peaks, SIZE);
    cout << "\n";

    // В отдельный
    // массив поместить все горные вершины в одной стране (страну вводить с
    // клавиатуры)
    cout << "Enter country name:\n";
    string s;
    cin >> s;
    int a_size;
    Peak* a = peaks_by_country(peaks, SIZE, s, a_size);
    print(a, a_size);

    delete[] a;
    cout << '\n';

    // Реализовать вывод отфильтрованных данных в виде оберточной
    // функции.

    print(peaks, SIZE, s, filter_by_country);
    cout << '\n';
    print(peaks, SIZE, 4000, filter_le_heigt);

    // ==========================================

    update_heights_from_file(peaks, SIZE, "task1.txt");
    print(peaks, SIZE);
    cout << '\n';

    task2_write(peaks, SIZE, "task2.bin");

    int b_size;
    Peak* b = task2_read("task2.bin", b_size);
    print(b, b_size);
}
