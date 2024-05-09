# project-of-product
#include <iostream>
#include <fstream>
#include <vector>
#include <string>
#include <iomanip>
#include <ctime>
#include <algorithm>
#include <time.h>

using namespace std;

// Structure to hold product information
struct Product {
    string name;
    int inventory;
    double price;
};

// Structure to hold customer information
struct Customer {
    string firstName;
    string lastName;
    string phoneNumber;
};

// Structure to hold order information
struct Order {
    Customer customer;
    vector<pair<string, int>> products; // pair<product name, quantity>
    double discount;
    double totalPrice;
    time_t timestamp;
};

// Function to register a new product
void registerProduct(const string& filename, vector<Product>& products) {
    string productName;
    int inventory;
    double price;

    cout << "Enter product name: ";
    getline(cin >> ws, productName);

    auto iter = find_if(products.begin(), products.end(), [&productName](Product& p) { return p.name == productName; });
    if (iter != products.end()) {
        // Product already exists, only update inventory
        cout << "Product already exists. Enter additional inventory: ";
        cin >> inventory;
        iter->inventory += inventory;
    } else {
        // New product, add to vector
        cout << "Enter inventory: ";
        cin >> inventory;
        cout << "Enter price: ";
        cin >> price;
        products.push_back({productName, inventory, price});
    }

    // Write products to file
    ofstream file(filename);
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return;
    }
    for (const auto& product : products) {
        file << product.name << " " << product.inventory << " " << fixed << setprecision(2) << product.price << endl;
    }
    file.close();

    cout << "Product registered successfully." << endl;
}
// Function to display all products and their inventory
void displayProducts(const string& filename) {
    ifstream file(filename);
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return;
    }

    cout << "Products:" << endl;
    string name;
    int inventory;
    double price;
    while (file >> name >> inventory >> price) {
        cout << "Name: " << name << ", Inventory: " << inventory << ", Price: $" << fixed << setprecision(2) << price << endl;
    }
    file.close();
}

// Function to read products from file into a vector
vector<Product> readProductsFromFile(const string& filename) {
    ifstream file(filename);
    vector<Product> products;
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return products;
    }

    string name;
    int inventory;
    double price;
    while (file >> name >> inventory >> price) {
        Product product{name, inventory, price};
        products.push_back(product);
    }
    file.close();
    return products;
}

// Function to update product inventory in file
void updateProductInventory(const string& filename, const string& productName, int newInventory) {
    ifstream fileIn(filename);
    ofstream fileOut("temp.txt");

    string name;
    int inventory;
    double price;
    while (fileIn >> name >> inventory >> price) {
        if (name == productName) {
            fileOut << name << " " << newInventory << " " << fixed << setprecision(2) << price << endl;
        } else {
            fileOut << name << " " << inventory << " " << fixed << setprecision(2) << price << endl;
        }
    }
    fileIn.close();
    fileOut.close();

    remove(filename.c_str());
    rename("temp.txt", filename.c_str());
}


const std::string currentDateTime() {
    time_t     now = time(0);
    struct tm  tstruct;
    char       buf[80];
    tstruct = *localtime(&now);
    strftime(buf, sizeof(buf), "%Y-%m-%d.%X", &tstruct);

    return buf;
}

time_t stringToTime(const string& timestampStr) {
    struct tm tm = {}; 
    char dot;
    istringstream iss(timestampStr);
    iss >> get_time(&tm, "%Y-%m-%d %H:%M:%S%c") >> dot;
    return mktime(&tm);
}


// Function to register a new order
void registerOrder(const string& filename, vector<Product>& products) {
    ofstream file(filename, ios::app);
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return;
    }

    Order order;
    cout << "Enter customer first name: ";
    getline(cin >> ws, order.customer.firstName);
    cout << "Enter customer last name: ";
    getline(cin >> ws, order.customer.lastName);
    cout << "Enter customer phone number: ";
    getline(cin >> ws, order.customer.phoneNumber);

    char addMore;
    do {
        string productName;
        int quantity;
        cout << "Enter product name: ";
        getline(cin >> ws, productName);
        auto iter = find_if(products.begin(), products.end(), [&productName](Product& p) { return p.name == productName; });
        if (iter == products.end()) {
            cout << "Product not found." << endl;
            continue;
        }

        cout << "Enter quantity: ";
        cin >> quantity;
        if (quantity > iter->inventory) {
            cout << "Insufficient inventory. Available quantity: " << iter->inventory << endl;
            continue;
        }

        order.products.push_back(make_pair(productName, quantity));

        // Update product inventory
        iter->inventory -= quantity;

        cout << "Add more products? (Y/N): ";
        cin >> addMore;
    } while (toupper(addMore) == 'Y');

    cout << "Enter discount percentage (0 for none): ";
    cin >> order.discount;

    // Calculate total price
    order.totalPrice = 0;
    for (const auto& product : order.products) {
        auto iter = find_if(products.begin(), products.end(), [&product](const Product& p) { return p.name == product.first; });
        order.totalPrice += iter->price * product.second;
    }
    order.totalPrice -= (order.totalPrice * order.discount) / 100;
    
    //order.timestamp = time(nullptr);

    // Write order to file
    file << order.customer.firstName << " " << order.customer.lastName << " " << order.customer.phoneNumber << " "
         << fixed << setprecision(2) << order.totalPrice << " " << currentDateTime() << endl;
    file.close();

    // Update product inventory in file
    ofstream productFile("products.txt");
    if (!productFile) {
        cerr << "Error: Unable to open product file." << endl;
        return;
    }
    for (const auto& product : products) {
        productFile << product.name << " " << product.inventory << " " << fixed << setprecision(2) << product.price << endl;
    }
    productFile.close();

    cout << "Order registered successfully." << endl;
}




// Function to display all orders
void displayOrders(const string& filename) {
    ifstream file(filename);
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return;
    }

    cout << "Orders:" << endl;
    string firstName, lastName, phoneNumber, timestampStr;
    double totalPrice;
    while (file >> firstName >> lastName >> phoneNumber >> totalPrice >> timestampStr) {
        // Parse string timestamp
        

        cout << "Customer: " << firstName << " " << lastName << ", Phone: " << phoneNumber
             << ", Total Price: $" << fixed << setprecision(2) << totalPrice
             << ", Date: " << timestampStr << endl;
    }
    file.close();
}


// Function to calculate turnover for a given date range
void calculateTurnover(const string& filename, const string& timestampStr1, const string& timestampStr2) {
    ifstream file(filename);
    if (!file) {
        cerr << "Error: Unable to open file." << endl;
        return;
    }

    time_t time1 = stringToTime(timestampStr1);
    time_t time2 = stringToTime(timestampStr2);

    //double secondes = difftime(time1, time2);
    double totalTurnover = 0;
    string firstName, lastName, phoneNumber;
    double totalPrice;
    time_t timestamp;
    
    while (file >> firstName >> lastName >> phoneNumber >> totalPrice >> timestamp) {
        //time_t timestamp = stringToTime(timestamp);
        
        //cout<<difftime(time2,timestamp)<<endl;
        if (difftime(time1,timestamp)>0 && difftime(time2,timestamp)>0) {
            totalTurnover += totalPrice;
        }
    }
    file.close();

    cout << "Total turnover for the specified date range: $" << fixed << setprecision(2) << totalTurnover << endl;
}

int main() {
    const string productFilename = "products.txt";
    const string orderFilename = "orders.txt";

    int choice;
    do {
        cout << "\nMenu:\n";
        cout << "1. Register a new product\n";
        cout << "2. Display all products\n";
        cout << "3. Register a new order\n";
        cout << "4. Display all orders\n";
        cout << "5. Calculate turnover\n";
        cout << "6. Exit\n";
        cout << "Enter your choice: ";
        cin >> choice;

        switch (choice) {
            case 1: {
                vector<Product> products = readProductsFromFile(productFilename);
                registerProduct(productFilename,products);
                break;
            }
            case 2:
                displayProducts(productFilename);
                break;
            case 3: {
                vector<Product> products = readProductsFromFile(productFilename);
                registerOrder(orderFilename, products);
                break;
            }
            case 4:
                displayOrders(orderFilename);
                break;
            case 5: {
                
                string startDate = "2023-03-25.23:10:13";
                string endDate = "2024-05-25.22:10:13";      
                calculateTurnover(orderFilename, startDate, endDate);
                break;
            }
            case 6:
                cout << "Exiting program...\n";
                break;
            default:
                cout << "Invalid choice. Please enter a number between 1 and 6.\n";
        }
    } while (choice != 6);

    return 0;
}
