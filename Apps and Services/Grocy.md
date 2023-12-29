# Install

```yaml
version: "2.1"
services:
  grocy:
    image: lscr.io/linuxserver/grocy:latest
    container_name: grocy
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Los_Angeles
    volumes:
      - /docker/appdata/grocy:/config
    ports:
      - 9283:80
    restart: unless-stopped
```

# Product Groups

```
Alcohol / Bar
Baked Goods
Beverages
Bulk
Canned & Jar Foods
Cereals
Cleaning Products
Coffee & Tea
Dairy & Eggs
Daycare
Frozen Meals
Health & Hygiene
Meat
Miscellaneous
Oils and Sauces
Pets
Produce
Snacks
Spices
Sweets
```
