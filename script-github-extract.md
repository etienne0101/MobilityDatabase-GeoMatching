# Script to extract Github comments and create a CSV
Fields: Github_id, date of comment

**All is done through Terminal**

### Create script
```
nano ~/Downloads/fetch_comments.sh
```

### Write script
```
#!/bin/bash

# Github token
GITHUB_TOKEN="insert_github_token_here"

# File creation
outfile=~/Downloads/commenters.csv
echo "pseudo,date" > "$outfile"

# Pagination
page=1
while true; do
  echo "Fetching page $page..."
  data=$(curl -s -H "Accept: application/vnd.github+json" \
              -H "Authorization: Bearer $GITHUB_TOKEN" \
              "https://api.github.com/repos/google/transit/issues/comments?per_page=100&page=$page")

  # Stop if no result
  if [ "$(echo "$data" | jq length)" -eq 0 ]; then
    break
  fi

  # Add comments to csv
  echo "$data" | jq -r '.[] | select(.user.login != null) | "\(.user.login),\(.created_at)"' >> "$outfile"

  page=$((page+1))
done

echo " result in $outfile"

```
Type 'ctrl+o', then 'Return', then 'ctrl+x'

### Make script executable
```
chmod +x ~/Downloads/fetch_comments.sh
```

### Launch script
```
~/Downloads/fetch_comments.sh
```


