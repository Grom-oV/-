#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <sstream>
#include <map>
#include <regex>


using namespace std;

struct Cargo {
    int id;
    string sender;
    string receiver;
    string cargoType;
    double weight;
    double distance;
    double cost;
    string status;
    string driverName;      
    string vehicleNumber;
};

class CargoManager {
private:
    vector<Cargo> cargos;
    int nextId;
    map<string, double> cityDistances; // Имитация базы данных расстояний

public:
    CargoManager() : nextId(1) {
        loadCargos();
        initializeCityDistances();
    }

    // Инициализация расстояний между городами
    void initializeCityDistances() {
        cityDistances["Minsk-Nigeria"] = 5000;
        cityDistances["Nigeria-Minsk"] = 5000;
        cityDistances["Turkia-Minsk"] = 1700;
        cityDistances["Moscow-Minsk"] = 670;
        cityDistances["Minsk-Kazan"] = 1400;
    }

    // Сохранение данных в файл
    void saveCargos() {
        ofstream file("cargos.txt");
        for (const auto& cargo : cargos) {
            file << cargo.id << ";" << cargo.sender << ";" << cargo.receiver << ";"
                << cargo.cargoType << ";" << cargo.weight << ";" << cargo.distance
                << ";" << cargo.cost << ";" << cargo.status << ";"
                << cargo.driverName << ";" << cargo.vehicleNumber << endl; 
        }
        file.close();
    }

    // Загрузка данных из файла
    void loadCargos() {
        ifstream file("cargos.txt");
        if (!file) return;

        cargos.clear();
        string line;
        while (getline(file, line)) {
            stringstream ss(line);
            Cargo c;
            string temp;
            getline(ss, temp, ';'); c.id = stoi(temp);
            getline(ss, c.sender, ';');
            getline(ss, c.receiver, ';');
            getline(ss, c.cargoType, ';');
            getline(ss, temp, ';'); c.weight = stod(temp);
            getline(ss, temp, ';'); c.distance = stod(temp);
            getline(ss, temp, ';'); c.cost = stod(temp);
            getline(ss, c.status, ';');
            getline(ss, c.driverName, ';');    
            getline(ss, c.vehicleNumber, ';'); 

            cargos.push_back(c);
            nextId = c.id + 1;
        }
        file.close();
    }


    // Расчет стоимости доставки
    double calculateCost(double weight, double distance) {
        return (distance * 100) + (weight * 50);
    }

    // Добавление нового груза
    void addCargo(string sender, string receiver, string cargoType, double weight, string driverName, string vehicleNumber) {
        string route = sender + "-" + receiver;
        double distance = cityDistances.count(route) ? cityDistances[route] : 1000; // Если нет данных, ставим 1000 км
        double cost = calculateCost(weight, distance);

        Cargo newCargo = { nextId++, sender, receiver, cargoType, weight, distance, cost, "В пути", driverName, vehicleNumber };
        cargos.push_back(newCargo);
        saveCargos();
        cout << "Груз добавлен с ID: " << newCargo.id << ", Расстояние: " << distance
            << " км, Стоимость: " << cost << " руб.<< " << ", Водитель: " << driverName
            << ", Машина: " << vehicleNumber << "\n";;
    }


    // Обновление статуса груза
    void updateCargoStatus(int id, string newStatus) {
        for (auto& cargo : cargos) {
            if (cargo.id == id) {
                cargo.status = newStatus;
                saveCargos();
                cout << "Статус груза ID " << id << " обновлен на: " << newStatus << "\n";
                return;
            }
        }
        cout << "Груз с ID " << id << " не найден.\n";
    }

    // Удаление груза
    void removeCargo(int id) {
        for (auto it = cargos.begin(); it != cargos.end(); ++it) {
            if (it->id == id) {
                cargos.erase(it);
                saveCargos();
                cout << "Груз с ID " << id << " удален.\n";
                return;
            }
        }
        cout << "Груз с ID " << id << " не найден.\n";
    }

    // Вывод всех грузов
    void displayCargos() {
        if (cargos.empty()) {
            cout << "Нет зарегистрированных грузов.\n";
            return;
        }
        cout << "Список грузов:\n";
        for (const auto& cargo : cargos) {
            cout << "ID: " << cargo.id << ", Отправитель: " << cargo.sender
                << ", Получатель: " << cargo.receiver << ", Тип: " << cargo.cargoType
                << ", Вес: " << cargo.weight << " кг, Расстояние: " << cargo.distance
                << " км, Стоимость: " << cargo.cost << " руб., Статус: " << cargo.status
                << ", Водитель: " << cargo.driverName << ", Машина: " << cargo.vehicleNumber << endl;
        }
    }

    // Поиск груза по ID
    void findCargo(int id) {
        for (const auto& cargo : cargos) {
            if (cargo.id == id) {
                cout << "Груз найден: ID " << cargo.id << ", Отправитель: " << cargo.sender
                    << ", Получатель: " << cargo.receiver << ", Тип: " << cargo.cargoType
                    << ", Вес: " << cargo.weight << " кг, Расстояние: " << cargo.distance
                    << " км, Стоимость: " << cargo.cost << " руб., Статус: " << cargo.status << endl;
                return;
            }
        }
        cout << "Груз с ID " << id << " не найден.\n";
    }
};


int main() {
    setlocale(LC_ALL, "Russian");
    CargoManager manager;
    int choice;

    do {
        cout << "\nМеню управления грузоперевозками:\n";
        cout << "1. Добавить груз\n";
        cout << "2. Показать все грузы\n";
        cout << "3. Найти груз по ID\n";
        cout << "4. Изменение статуса\n";
        cout << "5. Удолить груз \n";
        cout << "0. Выход\n";
        cout << "Выберите действие: ";
        cin >> choice;

        switch (choice) {
        case 1: {
            string sender, receiver, cargoType, driverName, vehicleNumber;
            double weight;
            cout << "Введите город отправителя: ";
            cin >> sender;
            cout << "Введите город получателя: ";
            cin >> receiver;
            cout << "Введите тип груза: ";
            cin >> cargoType;
            cout << "Введите вес (кг): ";
            cin >> weight;
            cout << "Имя водителя: ";
            cin >> driverName;
            regex platePattern(R"(^\d{4}[A-Z]{2}-\d$)");
            do {
                cout << "Введите номер машины (формат 1234AB-1): ";
                cin >> vehicleNumber;
                if (!regex_match(vehicleNumber, platePattern)) {
                    cout << "Некорректный формат. Попробуйте снова.\n";
                }
            } while (!regex_match(vehicleNumber, platePattern));

            manager.addCargo(sender, receiver, cargoType, weight, driverName, vehicleNumber);
            break;
        }
        case 2:
            manager.displayCargos();
            break;
        case 3: {
            int id;
            cout << "Введите ID груза: ";
            cin >> id;
            manager.findCargo(id);
            break;
        }

        case 4: {
            int id, statusChoice;
            cout << "Введите ID груза: ";
            cin >> id;

            cout << "Выберите новый статус:\n";
            cout << "1. В пути\n";
            cout << "2. Доставлен\n";
            cout << "3. Отменен\n";
            cout << "4. Ошибка\n";
            cout << "Ваш выбор: ";
            cin >> statusChoice;

            string status;
            switch (statusChoice) {
            case 1: status = "В пути"; break;
            case 2: status = "Доставлен"; break;
            case 3: status = "Отменен"; break;
            case 4: status = "Ошибка"; break;
            default:
                cout << "Некорректный выбор статуса.\n";
                continue;
            }

            manager.updateCargoStatus(id, status);
            break;
        }

        case 5:
        {
            int id;
            cout << "Введите ID для удаления: ";
            cin >> id;
            manager.removeCargo(id);
            break;
        }
        case 0:
            cout << "Выход из программы.\n";
            break;
        default:
            cout << "Некорректный ввод, попробуйте снова.\n";
        }
    } while (choice != 0);

    return 0;
}
