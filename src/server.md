# Potato Home Server

## Hardware
- Mainboard and CPU [Minisforum BD790i](https://store.minisforum.de/en/products/bd)
- Case [Fractal Design Torrent Nano](https://www.inet.se/produkt/6905120/fractal-design-torrent-nano-vit-tg)
- CPU Fan [Noctua NF-A12x25](https://www.inet.se/produkt/5323680/noctua-nf-a12x25-pwm-chromax-black)
- Power Supply [ATX PS2 750 W](https://www.inet.se/produkt/6903699/seasonic-focus-gx-750w)
- M2 SSDs [2x 4 TB](https://www.inet.se/produkt/4305453/wd-black-sn850x-4tb-gen-4)
- RAM [2x32 GB 6000 MHz](https://www.inet.se/produkt/5306277/kingston-64gb-2x32gb-ddr5-6000mhz-cl36-fury-beast-svart-amd-expo-intel-xmp-3-0)
- Hard Drives[]()


## Software
Plan:
- Mirror my desktop as much as possible for comfort
- Simpler services run as normal packages and systemd services
- Bigger complext stuff run in containters with podman
- Consider using nix for services


### Networking

Most services should only be accessible for clients connected with vpn or already on lan

#### WireGuard
Maybe use this?
[wg-easy](https://github.com/wg-easy/wg-easy)

#### Caddy

Add this rule to things that should only be available to wireguard connected clients
```toml
#https://caddyserver.com/docs/caddyfile/matchers#remote-ip
(local-only) {
  # private_ranges all private IPv4 and IPv6
  @denied not remote_ip private_ranges
  # abort @denied 
  respond @denied "<h1>No Potato</h1>" 403
}

# Example usage
http://mysite.example.com {    
    handle {
        import local-only

        # Your website handling logic goes here
        reverse_proxy 127.0.0.1:8080
    }
}
```
[Nextcloud Caddy Config](https://caddy.community/t/setting-up-nextcloud-behind-caddy/14787)
[Caddy local https](https://caddy.community/t/caddy-reverse-proxy-nextcloud-collabora-vaultwarden-with-local-https/12052)
