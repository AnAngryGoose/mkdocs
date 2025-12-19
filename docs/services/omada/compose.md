```yaml
### Omada Controller - TP-Link / Omada network controller ###

services:
  omada-controller:
    image: mbentley/omada-controller:latest
    container_name: omada-controller
    restart: always
    network_mode: host
    environment:
      - MANAGE_HTTP_PORT=8088
      - MANAGE_HTTPS_PORT=8043
      - PORTAL_HTTP_PORT=8088
      - PORTAL_HTTPS_PORT=8843
      - SHOW_SERVER_LOGS=true
      - SHOW_MONGODB_LOGS=false
      - TZ=America/Chicago
    # networks: 
    #   - proxy_net  
    volumes:   
      - omada_data:/opt/tplink/EAPController/data
      - omada_logs:/opt/tplink/EAPController/logs
      - omada_work:/opt/tplink/EAPController/work

volumes:
  omada_data:
    external: true
  omada_logs:
    external: true
  omada_work:
    external: true

# # --- Network Definitions --- #
# networks: 
#   proxy_net: 
#     external: true
```