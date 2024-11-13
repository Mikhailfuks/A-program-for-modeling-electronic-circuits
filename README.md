#include <iostream>
#include <vector>
#include <map>
#include <cmath>

using namespace std;

// Типы элементов схемы
enum class ElementType {
    RESISTOR,
    CAPACITOR,
    INDUCTOR,
    VOLTAGE_SOURCE,
    CURRENT_SOURCE,
};

// Базовый класс для элементов схемы
class CircuitElement {
public:
    CircuitElement(ElementType type, double value) : type(type), value(value) {}

    virtual double getVoltage(double current) const = 0;
    virtual double getCurrent(double voltage) const = 0;

    ElementType getType() const { return type; }
    double getValue() const { return value; }

private:
    ElementType type;
    double value;
};

// Резистор
class Resistor : public CircuitElement {
public:
    Resistor(double resistance) : CircuitElement(ElementType::RESISTOR, resistance) {}

    double getVoltage(double current) const override {
        return current * getValue();
    }

    double getCurrent(double voltage) const override {
        return voltage / getValue();
    }
};

// Конденсатор
class Capacitor : public CircuitElement {
public:
    Capacitor(double capacitance) : CircuitElement(ElementType::CAPACITOR, capacitance) {}

    double getVoltage(double current) const override {
        // Для моделирования конденсатора нужна временная составляющая
        // Предположим, что это статическая схема
        return 0.0; 
    }

    double getCurrent(double voltage) const override {
        // Аналогично, нам нужно знать изменение напряжения во времени
        return 0.0;
    }
};

// Индуктор
class Inductor : public CircuitElement {
public:
    Inductor(double inductance) : CircuitElement(ElementType::INDUCTOR, inductance) {}

    double getVoltage(double current) const override {
        // Для моделирования индуктора нужна временная составляющая
        // Предположим, что это статическая схема
        return 0.0; 
    }

    double getCurrent(double voltage) const override {
        // Аналогично, нам нужно знать изменение тока во времени
        return 0.0;
    }
};

// Источник напряжения
class VoltageSource : public CircuitElement {
public:
    VoltageSource(double voltage) : CircuitElement(ElementType::VOLTAGE_SOURCE, voltage) {}

    double getVoltage(double current) const override {
        return getValue();
    }

    double getCurrent(double voltage) const override {
        // Ток определяется внешней цепью
        return 0.0; 
    }
};

// Источник тока
class CurrentSource : public CircuitElement {
public:
    CurrentSource(double current) : CircuitElement(ElementType::CURRENT_SOURCE, current) {}

    double getVoltage(double current) const override {
        // Напряжение определяется внешней цепью
        return 0.0; 
    }

    double getCurrent(double voltage) const override {
        return getValue();
    }
};

// Класс для моделирования схемы
class Circuit {
public:
    void addElement(CircuitElement* element) {
        elements.push_back(element);
    }

    // Метод для решения схемы методом узловых потенциалов
    void solve() {
        // 1. Создаем список узлов
        vector<int> nodes;
        for (auto element : elements) {
            if (element->getType() != ElementType::VOLTAGE_SOURCE) {
                nodes.push_back(1); // Предполагаем, что каждый элемент подключен к узлу 1
            }
        }

        // 2. Создаем матрицу проводимости G
        map<pair<int, int>, double> G;
        for (auto element : elements) {
            if (element->getType() == ElementType::RESISTOR) {
                Resistor* resistor = dynamic_cast<Resistor*>(element);
                G[{1, 1}] += 1.0 / resistor->getValue(); // Предполагаем, что оба конца резистора подключены к узлу 1
            }
        }

        // 3. Создаем вектор независимых источников I
        vector<double> I(nodes.size(), 0.0);
        for (auto element : elements) {
            if (element->getType() == ElementType::CURRENT_SOURCE) {
                CurrentSource* currentSource = dynamic_cast<CurrentSource*>(element);
                I[0] += currentSource->getValue(); // Предполагаем, что ток источника втекает в узел 1
            }
        }

        // 4. Решаем систему уравнений G * V = I
        // Используем упрощенный метод, т.к. схема простая
        double V = I[0] / G[{1, 1}]; // Решаем для узла 1

        // 5. Выводим результаты
        cout << "Напряжение на узле 1: " << V << " В" << endl;

        for (auto element : elements) {
            if (element->getType() == ElementType::RESISTOR) {
                Resistor* resistor = dynamic_cast<Resistor*>(element);
                cout << "Ток через резистор: " << resistor->getCurrent(V) << " А" << endl;
            }
        }
    }

private:
    vector<CircuitElement*> elements;
};

int main() {
    Circuit circuit;

    // Добавляем элементы в схему
    circuit.addElement(new Resistor(100.0)); // Резистор 100 Ом
    circuit.addElement(new VoltageSource(12.0)); // Источник напряжения 12 В
    circuit.addElement(new CurrentSource(0.1)); // Источник тока 0.1 А

    // Решаем схему
    circuit.solve();

    return 0;
}
