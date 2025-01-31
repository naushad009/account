# account
import json
from datetime import datetime

# File names
MASTER_FILE = 'master.json'
TRANSACTION_FILE = 'transaction.json'

# Initialize files if they don't exist
for file in [MASTER_FILE, TRANSACTION_FILE]:
    try:
        with open(file, 'x') as f:
            f.write('[]')  # Initialize with an empty list
    except FileExistsError:
        pass

# Helper functions
def load_data(filename):
    """Load data from a JSON file."""
    try:
        with open(filename, 'r') as file:
            return json.load(file)
    except (json.JSONDecodeError, FileNotFoundError):
        return []

def save_data(filename, data):
    """Save data to a JSON file."""
    with open(filename, 'w') as file:
        json.dump(data, file, indent=4)

def account_exists(acno, master_data):
    """Check if an account exists."""
    return any(account["AccountNo"] == acno for account in master_data)

def get_account(acno, master_data):
    """Get account details by account number."""
    for account in master_data:
        if account["AccountNo"] == acno:
            return account
    return None

# Main application
def bank_menu():
    while True:
        print("\n=== Kangal Bank Menu ===")
        print("1. Open Account")
        print("2. Deposit Amount")
        print("3. Withdraw Amount")
        print("4. Show Balance")
        print("5. Exit")

        try:
            choice = int(input("Enter your choice: "))
        except ValueError:
            print("Invalid input. Please enter a number between 1 and 5.")
            continue

        master_data = load_data(MASTER_FILE)
        transaction_data = load_data(TRANSACTION_FILE)
        today_date = datetime.now().strftime("%d-%m-%Y")

        if choice == 1:
            # Open Account
            try:
                acno = int(input("Enter Account Number: "))
                cname = input("Enter Customer Name: ").strip()
                amt = float(input("Enter Initial Deposit Amount: "))
                if amt <= 0:
                    raise ValueError("Initial deposit must be greater than 0.")
            except ValueError as e:
                print(f"Error: {e}")
                continue

            if account_exists(acno, master_data):
                print("Account number already exists.")
            else:
                new_account = {"AccountNo": acno, "Name": cname, "Date": today_date, "Balance": amt}
                master_data.append(new_account)
                transaction_data.append({"AccountNo": acno, "Date": today_date, "Type": "Deposit", "Amount": amt, "Balance": amt})
                save_data(MASTER_FILE, master_data)
                save_data(TRANSACTION_FILE, transaction_data)
                print("Account successfully created!")

        elif choice == 2:
            # Deposit Amount
            try:
                acno = int(input("Enter Account Number: "))
                amt = float(input("Enter Deposit Amount: "))
                if amt <= 0:
                    raise ValueError("Deposit amount must be greater than 0.")
            except ValueError as e:
                print(f"Error: {e}")
                continue

            account = get_account(acno, master_data)
            if account:
                account["Balance"] += amt
                transaction_data.append({"AccountNo": acno, "Date": today_date, "Type": "Deposit", "Amount": amt, "Balance": account["Balance"]})
                save_data(MASTER_FILE, master_data)
                save_data(TRANSACTION_FILE, transaction_data)
                print("Amount deposited successfully!")
            else:
                print("Account not found.")

        elif choice == 3:
            # Withdraw Amount
            try:
                 acno = int(input("Enter Account Number: "))
                 amt = float(input("Enter Withdrawal Amount: "))
                 if amt <= 0:
                     raise ValueError("Withdrawal amount must be greater than 0.")
            except ValueError as e:
                 print(f"Error: {e}")
                 continue

            account = get_account(acno, master_data)
            if account:
                if account["Balance"] >= amt:
                     account["Balance"] -= amt
                     transaction_data.append({"AccountNo": acno, "Date": today_date, "Type": "Withdraw", "Amount": amt, "Balance": account["Balance"]})
                     save_data(MASTER_FILE, master_data)
                     save_data(TRANSACTION_FILE, transaction_data)
                     print("Amount withdrawn successfully!")
                else:
                     print("Insufficient funds.")
            else:
                print("Account not found.")
        
        elif choice == 4:
            # Show Balance
            try:
                acno = int(input("Enter Account Number: "))
            except ValueError:
                print("Invalid account number.")
                continue

            account = get_account(acno, master_data)
            if account:
                print("\n=== Account Details ===")
                print(f"Account Number: {account['AccountNo']}")
                print(f"Customer Name: {account['Name']}")
                print(f"Balance: {account['Balance']}")
            else:
                print("Account not found.")

        elif choice == 5:
            print("Thank you for using Kangal Bank. Goodbye!")
            break

        else:
            print("Invalid choice. Please select a valid option.")

# Run the application
if __name__ == "__main__":
    bank_menu()
