# Multi-architecture demo image for Skelz
FROM debian:trixie-slim

# Add metadata labels
LABEL org.opencontainers.image.title="Skelz Demo Image"
LABEL org.opencontainers.image.description="Demo container image for Skelz - decentralized container registry on Solana"
LABEL org.opencontainers.image.vendor="Kevred Labs"
LABEL org.opencontainers.image.source="https://github.com/kevredlabs/skelz-hackathon"
LABEL org.opencontainers.image.version="1.0.0"
RUN apt-get update && apt-get install -y --no-install-recommends figlet && rm -rf /var/lib/apt/lists/*
ENTRYPOINT ["figlet"]
CMD ["Hello SuperteamFR !"]