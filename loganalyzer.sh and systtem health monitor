q1 answer:
#!/bin/bash

log_file="$1"
timestamp=$(date +"%Y%m%d_%H%M%S")
report_file="log_analysis_${timestamp}.txt"

echo "===== LOG FILE ANALYSIS REPORT ====="
echo "File: $log_file"
echo -n "Analyzed on: "; date
echo -n "Size: "; du -h "$log_file" | cut -f1; echo " ($(stat -c%s "$log_file") bytes)"

errors=$(grep -c "ERROR" "$log_file")
warnings=$(grep -c "WARNING" "$log_file")
infos=$(grep -c "INFO" "$log_file")

echo ""
echo "MESSAGE COUNTS:"
echo "ERROR: $errors messages"
echo "WARNING: $warnings messages"
echo "INFO: $infos messages"

echo ""
echo "TOP 5 ERROR MESSAGES:"
grep "ERROR" "$log_file" | cut -d']' -f2- | sed 's/^ //' | sort | uniq -c | sort -nr | head -5 | while read count msg; do
  echo "$count - $msg"
done

first_error=$(grep "ERROR" "$log_file" | head -1)
last_error=$(grep "ERROR" "$log_file" | tail -1)

echo ""
echo "ERROR TIMELINE:"
echo "First error: $first_error"
echo "Last error:  $last_error"

echo ""
echo "Error frequency by hour:"
for range in "00-04" "04-08" "08-12" "12-16" "16-20" "20-24"; do
  start=${range%-*}
  end=${range#*-}
  count=$(grep "ERROR" "$log_file" | awk -F'[][]' '{print $2}' | cut -d: -f1 | awk -v s=$start -v e=$end '$1 >= s && $1 < e' | wc -l)
  bar=$(printf "%*s" $((count/2)) "" | tr ' ' '█')
  echo "$range: $bar ($count)"
done

{
  echo "===== LOG FILE ANALYSIS REPORT ====="
  echo "File: $log_file"
  echo "Analyzed on: $(date)"
  echo "Size: $(du -h "$log_file" | cut -f1) ($(stat -c%s "$log_file") bytes)"
  echo ""
  echo "MESSAGE COUNTS:"
  echo "ERROR: $errors messages"
  echo "WARNING: $warnings messages"
  echo "INFO: $infos messages"
  echo ""
  echo "TOP 5 ERROR MESSAGES:"
  grep "ERROR" "$log_file" | cut -d']' -f2- | sed 's/^ //' | sort | uniq -c | sort -nr | head -5
  echo ""
  echo "ERROR TIMELINE:"
  echo "First error: $first_error"
  echo "Last error:  $last_error"
  echo ""
  echo "Error frequency by hour:"
  for range in "00-04" "04-08" "08-12" "12-16" "16-20" "20-24"; do
    start=${range%-*}
    end=${range#*-}
    count=$(grep "ERROR" "$log_file" | awk -F'[][]' '{print $2}' | cut -d: -f1 | awk -v s=$start -v e=$end '$1 >= s && $1 < e' | wc -l)
    bar=$(printf "%*s" $((count/2)) "" | tr ' ' '█')
    echo "$range: $bar ($count)"
  done
} > "$report_file"

echo ""
echo "Report saved to: $report_file"

question 2:
#!/bin/bash

refresh=3
filter="all"
log_file="health_alerts.log"

while true; do
  clear
  hostname=$(hostname)
  now=$(date)
  uptime_val=$(uptime -p)
  cpu=$(top -bn1 | grep "Cpu(s)" | awk '{print 100 - $8}')
  mem_used=$(free -m | awk '/Mem:/ {print $3}')
  mem_total=$(free -m | awk '/Mem:/ {print $2}')
  mem_perc=$((100 * mem_used / mem_total))
  disk_root=$(df / | awk 'NR==2 {print $5}' | tr -d '%')
  net_in=$(cat /proc/net/dev | grep eth0 | awk '{print $2}')
  net_out=$(cat /proc/net/dev | grep eth0 | awk '{print $10}')
  load=$(uptime | awk -F'load average:' '{print $2}')

  echo "╔════════════ SYSTEM HEALTH MONITOR v1.0 ════════════╗"
  echo " [R]efresh rate: ${refresh}s ║ Hostname: $hostname          Date: $now"
  echo " [F]ilter: $filter ║ Uptime: $uptime_val"
  echo " [Q]uit ╚══════════════════════════════════════════════════════════════════════╝"

  echo -n " CPU USAGE: $cpu% "
  bar=$(printf "%*s" $((cpu / 2)) "" | tr ' ' '█')
  echo -n "$bar"
  if (( $(echo "$cpu > 80" | bc -l) )); then
    echo " [CRITICAL]"
    echo "[$(date +%T)] CPU usage exceeded 80% ($cpu%)" >> "$log_file"
  elif (( $(echo "$cpu > 60" | bc -l) )); then
    echo " [WARNING]"
  else
    echo " [OK]"
  fi

  echo -n " MEMORY: ${mem_used}MB/${mem_total}MB ($mem_perc%) "
  bar=$(printf "%*s" $((mem_perc / 2)) "" | tr ' ' '█')
  echo -n "$bar"
  if (( mem_perc > 75 )); then
    echo " [WARNING]"
    echo "[$(date +%T)] Memory usage exceeded 75% (${mem_perc}%)" >> "$log_file"
  else
    echo " [OK]"
  fi

  echo -n " DISK USAGE: / : ${disk_root}% "
  bar=$(printf "%*s" $((disk_root / 2)) "" | tr ' ' '█')
  echo -n "$bar"
  if (( disk_root > 75 )); then
    echo " [WARNING]"
    echo "[$(date +%T)] Disk usage on / exceeded 75% (${disk_root}%)" >> "$log_file"
  else
    echo " [OK]"
  fi

  echo -n " NETWORK: eth0 (in): $((net_in/1024/1024)) MB/s "
  echo " | eth0 (out): $((net_out/1024/1024)) MB/s"

  echo " LOAD AVERAGE:$load"

  echo ""
  echo " RECENT ALERTS:"
  tail -n 5 "$log_file" 2>/dev/null

  read -t $refresh -n 1 key
  if [[ $key == "q" ]]; then
    break
  elif [[ $key == "r" ]]; then
    read -p "Enter new refresh rate (seconds): " refresh
  elif [[ $key == "f" ]]; then
    read -p "Filter (all/cpu/mem/disk): " filter
  fi
done
