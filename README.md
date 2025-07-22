import hashlib
import os
import json

# Function to calculate hash of a file
def calculate_hash(filepath):
    sha256_hash = hashlib.sha256()
    try:
        with open(filepath, "rb") as f:
            for byte_block in iter(lambda: f.read(4096), b""):
                sha256_hash.update(byte_block)
        return sha256_hash.hexdigest()
    except FileNotFoundError:
        return None

# Function to save hash records to a file
def save_hashes(directory, output_file="hashes.json"):
    hash_dict = {}
    for root, _, files in os.walk(directory):
        for filename in files:
            path = os.path.join(root, filename)
            file_hash = calculate_hash(path)
            if file_hash:
                hash_dict[path] = file_hash
    with open(output_file, "w") as f:
        json.dump(hash_dict, f, indent=4)
    print(f"[+] Hashes saved to {output_file}")

# Function to check file integrity
def check_integrity(hash_file="hashes.json"):
    with open(hash_file, "r") as f:
        old_hashes = json.load(f)

    for path, old_hash in old_hashes.items():
        current_hash = calculate_hash(path)
        if current_hash is None:
            print(f"[!] File missing: {path}")
        elif current_hash != old_hash:
            print(f"[!] File changed: {path}")
        else:
            print(f"[OK] File unchanged: {path}")

# Main menu
def main():
    print("=== File Integrity Checker ===")
    print("1. Generate and Save Hashes")
    print("2. Check File Integrity")
    choice = input("Enter your choice (1/2): ")

    if choice == "1":
        directory = input("Enter directory path to monitor: ")
        save_hashes(directory)
    elif choice == "2":
        hash_file = input("Enter hash file (default: hashes.json): ") or "hashes.json"
        check_integrity(hash_file)
    else:
        print("Invalid choice!")

if __name__ == "__main__":
    main()
