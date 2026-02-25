# SERVER-MONITORING-AND-SECURITY-TOOL
BASH
#!/bin/bash

#############################################################
#               SYSTEM GUARDIAN v2.0                       #
#        Advanced Server Monitoring Tool (Bash)           #
#        Author: Braj Kishor Das                           #
#############################################################

LOG_FILE="guardian.log"
REPORT_FILE="system_report_$(date +%Y%m%d_%H%M%S).txt"
ALERT_THRESHOLD_CPU=80
ALERT_THRESHOLD_RAM=80
ALERT_THRESHOLD_DISK=80

# ---------- Color Codes ----------
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
CYAN='\033[0;36m'
NC='\033[0m'

# ---------- Logging Function ----------
log_action() {
    echo "$(date '+%Y-%m-%d %H:%M:%S') - $1" >> $LOG_FILE
}

# ---------- System Information ----------
system_info() {
    echo -e "${CYAN}===== SYSTEM INFORMATION =====${NC}"
    echo "Hostname: $(hostname)"
    echo "Uptime:"
    uptime
    echo
    echo "Kernel Version:"
    uname -r
    echo
    log_action "Viewed system information"
}

# ---------- CPU Monitoring ----------
cpu_usage() {
    usage=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')
    echo -e "${CYAN}CPU Usage:${NC} $usage %"
    
    if (( $(echo "$usage > $ALERT_THRESHOLD_CPU" | bc -l) )); then
        echo -e "${RED}WARNING: High CPU Usage!${NC}"
        log_action "High CPU usage detected: $usage%"
    fi
}

# ---------- RAM Monitoring ----------
ram_usage() {
    usage=$(free | awk '/Mem/ {printf("%.2f"), $3/$2 * 100.0}')
    echo -e "${CYAN}RAM Usage:${NC} $usage %"
    
    if (( $(echo "$usage > $ALERT_THRESHOLD_RAM" | bc -l) )); then
        echo -e "${RED}WARNING: High RAM Usage!${NC}"
        log_action "High RAM usage detected: $usage%"
    fi
}

# ---------- Disk Monitoring ----------
disk_usage() {
    echo -e "${CYAN}Disk Usage:${NC}"
    df -h | awk 'NR>1 {print $5 " " $1}' | while read output;
    do
        usep=$(echo $output | awk '{print $1}' | cut -d'%' -f1 )
        partition=$(echo $output | awk '{print $2}' )
        if [ $usep -ge $ALERT_THRESHOLD_DISK ]; then
            echo -e "${RED}Partition \"$partition\" almost full ($usep%)${NC}"
            log_action "Disk alert: $partition at $usep%"
        else
            echo -e "${GREEN}$partition is OK ($usep%)${NC}"
        fi
    done
}

# ---------- Top Processes ----------
top_processes() {
    echo -e "${CYAN}Top 5 Memory Consuming Processes:${NC}"
    ps aux --sort=-%mem | head -6
    log_action "Viewed top processes"
}

# ---------- Open Ports Scanner ----------
scan_ports() {
    echo -e "${CYAN}Open Ports:${NC}"
    netstat -tuln
    log_action "Scanned open ports"
}

# ---------- Failed Login Attempts ----------
failed_logins() {
    echo -e "${CYAN}Failed Login Attempts:${NC}"
    grep "Failed password" /var/log/auth.log 2>/dev/null | tail -5
    log_action "Checked failed login attempts"
}

# ---------- Generate Full Report ----------
generate_report() {
    {
        echo "=========== SYSTEM REPORT ==========="
        date
        echo
        system_info
        cpu_usage
        ram_usage
        disk_usage
        top_processes
    } > $REPORT_FILE

    echo -e "${GREEN}Report Generated: $REPORT_FILE${NC}"
    log_action "Generated full system report"
}

# ---------- Main Menu ----------
menu() {
while true
do
    echo
    echo -e "${YELLOW}
# OUTPUT
