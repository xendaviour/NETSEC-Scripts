#!/bin/bash

# Function to get user input
get_target() {
    echo "Select input method:"
    echo "1. Single IP"
    echo "2. IP List (file)"
    echo "3. IP Range/Subnet"
    read -p "Enter choice (1/2/3): " choice

    case $choice in
        1)
            read -p "Enter a single IP: " target
            ;;
        2)
            read -p "Enter the file path containing IPs: " ip_file
            if [[ ! -f "$ip_file" ]]; then
                echo "File not found!"
                exit 1
            fi
            target=$(cat "$ip_file" | xargs)
            ;;
        3)
            read -p "Enter the IP range or subnet (e.g., 192.168.1.0/24): " target
            ;;
        *)
            echo "Invalid choice!"
            exit 1
            ;;
    esac
}

# Function to run Nmap and process results
run_nmap() {
    echo "Running Nmap scan..."
    nmap -O -p- --max-retries 1 --osscan-guess $target -oX nmap_output.xml > /dev/null

    echo "Processing results..."
    csv_output="nmap_os_results.csv"
    xlsx_output="nmap_os_results.xlsx"

    echo "IP Address,MAC Address,OS Version,Open Ports" > "$csv_output"

    # Extract and parse XML data properly
    while IFS= read -r ip; do
        mac=$(xmllint --xpath "string(//host[address[@addr='$ip']]/address[@addrtype='mac']/@addr)" nmap_output.xml 2>/dev/null)
        os=$(xmllint --xpath "string(//host[address[@addr='$ip']]/os/osmatch[1]/@name)" nmap_output.xml 2>/dev/null)
        ports=$(xmllint --xpath "//host[address[@addr='$ip']]/ports/port[state/@state='open']/@portid" nmap_output.xml 2>/dev/null | tr '\n' ' ' | sed 's/ $//')

        [[ -z "$mac" ]] && mac="Unknown"
        [[ -z "$os" ]] && os="Unknown OS"
        [[ -z "$ports" ]] && ports="None"

        echo "$ip,$mac,$os,$ports" >> "$csv_output"
    done < <(xmllint --xpath "//host/address[@addrtype='ipv4']/@addr" nmap_output.xml 2>/dev/null | grep -oP '(?<=addr=")[^"]+')

    echo "CSV results saved to $csv_output"

    # Convert CSV to XLSX using Python
    python3 - <<EOF
import pandas as pd

df = pd.read_csv("$csv_output")
df.to_excel("$xlsx_output", index=False)

print("XLSX results saved to $xlsx_output")
EOF
}

# Check if dependencies are installed
check_dependencies() {
    if ! command -v xmllint &> /dev/null; then
        echo "xmllint is not installed! Install 'libxml2-utils' and try again."
        exit 1
    fi

    if ! command -v python3 &> /dev/null; then
        echo "Python3 is not installed! Install it and try again."
        exit 1
    fi

    if ! python3 -c "import pandas" 2>/dev/null; then
        echo "Pandas library not found! Installing..."
        pip3 install pandas openpyxl
    fi
}

# Main execution
check_dependencies
get_target
run_nmap
