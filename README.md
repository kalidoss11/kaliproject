#include <iostream>
#include <vector>
#include <string>
#include <fstream>
#include <iomanip>
#include <algorithm>

using namespace std;

// Class representing a single financial transaction
class Transaction {
public:
    int id;
    string date;
    string category;
    string description;
    double amount;
    bool isExpense;

    Transaction(int i, string d, string c, string desc, double amt, bool exp)
        : id(i), date(d), category(c), description(desc), amount(amt), isExpense(exp) {}

    void display() const {
        cout << left << setw(5) << id 
             << setw(12) << date 
             << setw(15) << category 
             << setw(25) << description 
             << setw(10) << (isExpense ? "-" : "+") << amount << endl;
    }
};

// Manager class to handle logic
class FinanceTracker {
private:
    vector<Transaction> transactions;
    const string filename = "ledger.txt";
    int nextId = 1;

    void loadFromFile() {
        ifstream file(filename);
        if (!file.is_open()) return;

        int id;
        string date, cat, desc;
        double amt;
        bool exp;

        while (file >> id >> date >> cat >> ws && getline(file, desc, '|') && file >> amt >> exp) {
            transactions.emplace_back(id, date, cat, desc, amt, exp);
            nextId = max(nextId, id + 1);
        }
        file.close();
    }

    void saveToFile() {
        ofstream file(filename);
        for (const auto& t : transactions) {
            file << t.id << " " << t.date << " " << t.category << " " 
                 << t.description << "|" << t.amount << " " << t.isExpense << endl;
        }
        file.close();
    }

public:
    FinanceTracker() {
        loadFromFile();
    }

    void addTransaction() {
        string date, cat, desc;
        double amt;
        int type;

        cout << "\n--- Add New Transaction ---\n";
        cout << "Enter Date (YYYY-MM-DD): "; cin >> date;
        cout << "Enter Category: "; cin >> cat;
        cout << "Enter Description: "; cin.ignore(); getline(cin, desc);
        cout << "Enter Amount: "; cin >> amt;
        cout << "Type (1 for Expense, 2 for Income): "; cin >> type;

        transactions.emplace_back(nextId++, date, cat, desc, amt, (type == 1));
        saveToFile();
        cout << "Transaction recorded successfully!\n";
    }

    void showSummary() {
        double totalIncome = 0, totalExpense = 0;

        cout << "\n" << string(70, '=') << endl;
        cout << left << setw(5) << "ID" << setw(12) << "Date" << setw(15) << "Category" 
             << setw(25) << "Description" << setw(10) << "Amount" << endl;
        cout << string(70, '-') << endl;

        for (const auto& t : transactions) {
            t.display();
            if (t.isExpense) totalExpense += t.amount;
            else totalIncome += t.amount;
        }

        cout << string(70, '=') << endl;
        cout << "Total Income:  $" << fixed << setprecision(2) << totalIncome << endl;
        cout << "Total Expense: $" << totalExpense << endl;
        cout << "Net Balance:   $" << (totalIncome - totalExpense) << endl;
        cout << string(70, '=') << endl;
    }

    void deleteTransaction() {
        int id;
        cout << "Enter Transaction ID to delete: "; cin >> id;
        
        auto it = remove_if(transactions.begin(), transactions.end(), [id](const Transaction& t) {
            return t.id == id;
        });

        if (it != transactions.end()) {
            transactions.erase(it, transactions.end());
            saveToFile();
            cout << "Deleted successfully.\n";
        } else {
            cout << "ID not found.\n";
        }
    }
};

int main() {
    FinanceTracker tracker;
    int choice;

    do {
        cout << "\n--- BUDGET TRACKER PRO ---" << endl;
        cout << "1. View Ledger\n2. Add Transaction\n3. Delete Transaction\n4. Exit\n";
        cout << "Choice: ";
        cin >> choice;

        switch (choice) {
            case 1: tracker.showSummary(); break;
            case 2: tracker.addTransaction(); break;
            case 3: tracker.deleteTransaction(); break;
            case 4: cout << "Goodbye!\n"; break;
            default: cout << "Invalid option.\n";
        }
    } while (choice != 4);

    return 0;
}
