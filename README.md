# login_detector.sh
#!/bin/bash

LOGFILE="auth.log"
REPORT="security_report.txt"

echo "Linux Security Login Report"
echo "==========================="

# Step 1: Check whether the log file exists
if [ ! -f "$LOGFILE" ]; then
    echo "Error: Log file $LOGFILE not found!"
    exit 1
fi

# Step 2: Count failed login attempts
FAILED_COUNT=$(grep -c "Failed password" "$LOGFILE")
echo "Failed logins: $FAILED_COUNT"

# Step 3: Count successful login attempts
SUCCESS_COUNT=$(grep -c "Accepted password" "$LOGFILE")
echo "Successful logins: $SUCCESS_COUNT"

# Step 4: Identify IP addresses with failed logins
FAILED_IPS=$(grep "Failed password" "$LOGFILE" | awk '{print $NF}' | sort | uniq -c)

# Step 5: Generate security alerts
echo "Suspicious IPs:" 
echo "$FAILED_IPS" | while read COUNT IP; do
    if [ "$COUNT" -ge 3 ]; then
        echo "ALERT: $IP has $COUNT failed attempts"
    fi
done

# Step 6: Save the results to a report
echo "Security Login Report" > "$REPORT"
echo "=====================" >> "$REPORT"
echo "" >> "$REPORT"

echo "Failed logins: $FAILED_COUNT" >> "$REPORT"
echo "Successful logins: $SUCCESS_COUNT" >> "$REPORT"
echo "" >> "$REPORT"

echo "Failed attempts by IP address:" >> "$REPORT"
echo "$FAILED_IPS" | while read COUNT IP; do
    if [ "$COUNT" -ge 3 ]; then
        echo -e "$IP\t$COUNT attempts\tALERT" >> "$REPORT"
    else
        echo -e "$IP\t$COUNT attempts" >> "$REPORT"
    fi
done

echo "" >> "$REPORT"
echo "Report saved to $REPORT"
