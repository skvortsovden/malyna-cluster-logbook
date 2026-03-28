# monitoring

## prometheus

### how to get directory sizes on linux machine as metrics in grafana

I want to see what files/folders are taking up the most space on my nodes. This script outputs Prometheus textfile metrics for top-level directory sizes, which can be scraped by node_exporter.

```bash
sudo tee /usr/local/bin/du-metrics.sh > /dev/null <<'EOF'
#!/bin/bash
# Outputs Prometheus textfile metrics for top-level directory sizes
OUTPUT=/var/lib/node_exporter/textfile/du.prom

mkdir -p "$(dirname "$OUTPUT")"

echo "# HELP node_directory_size_bytes Disk usage of directories in bytes" > "$OUTPUT"
echo "# TYPE node_directory_size_bytes gauge" >> "$OUTPUT"

# Scan root-level dirs; add more paths as needed
for dir in /home /var /opt /usr /etc /tmp; do
  [ -d "$dir" ] || continue
  bytes=$(du -sb "$dir" 2>/dev/null | awk '{print $1}')
  echo "node_directory_size_bytes{path=\"${dir}\",host=\"$(hostname)\"} ${bytes}" >> "$OUTPUT"
done
EOF
```

```bash
sudo chmod +x /usr/local/bin/du-metrics.sh
```

Then add a cron job to run this script every 5 minutes:

```bash
sudo tee /etc/cron.d/du-metrics > /dev/null <<'EOF'
*/5 * * * * root /usr/local/bin/du-metrics.sh
EOF
```

This will create a file `/var/lib/node_exporter/textfile/du.prom` with metrics like:

```# HELP node_directory_size_bytes Disk usage of directories in bytes
# TYPE node_directory_size_bytes gauge
node_directory_size_bytes{path="/home",host="my-node"} 123456789
node_directory_size_bytes{path="/var",host="my-node"} 987654321
```

Then create a Prometheus query to see the top 10 largest directories on the node:

```
topk(10,
  node_directory_size_bytes{host="lohyna"}
)
```