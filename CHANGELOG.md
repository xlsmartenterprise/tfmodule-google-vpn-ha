# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2025-12-13

### Added

#### Core VPN Infrastructure
- HA VPN gateway resource with automatic high availability configuration
- External VPN gateway resource for connecting to on-premises or third-party VPNs
- Support for GCP-to-GCP VPN connections via `peer_gcp_gateway`
- Automatic Cloud Router creation with BGP configuration
- Support for using existing Cloud Router via `router_name` parameter
- Support for using existing VPN gateway via `vpn_gateway_self_link`

#### Tunnel Management
- Multiple VPN tunnel support with individual BGP session configuration
- Automatic shared secret generation using random provider
- Custom shared secret support per tunnel
- IKE version selection (IKEv1 or IKEv2) per tunnel
- Dual interface support for high availability (interface 0 and 1)

#### BGP Configuration
- BGP router configuration with customizable ASN
- BGP keepalive interval configuration (default: 20 seconds)
- Global route priority setting (default: 1000)
- Per-tunnel route priority override via `bgp_peer_options`
- BGP session range configuration for peer IP addressing
- Router interface creation for each VPN tunnel

#### Advanced BGP Features
- Custom BGP advertisement modes (DEFAULT or CUSTOM)
- Advertised groups configuration (e.g., ALL_SUBNETS)
- Custom IP range advertisement with descriptions
- Per-tunnel BGP advertisement configuration override
- Import policy support for BGP route filtering
- Export policy support for BGP route filtering
- Router-level and tunnel-level advertisement controls

#### Gateway Configuration
- External VPN gateway with configurable redundancy types:
  - SINGLE_IP_INTERNALLY_REDUNDANT
  - TWO_IPS_REDUNDANCY
  - FOUR_IPS_REDUNDANCY
- Multiple interface support for external gateways
- Custom external gateway naming and descriptions
- Stack type configuration (IPv4_ONLY, IPv4_IPv6, IPv6_ONLY)

#### Interconnect Integration
- IPsec-encrypted Cloud Interconnect support via VPN interfaces
- Interconnect attachment URL configuration
- Multiple attachment support for redundancy

#### Resource Management
- Flexible VPN gateway creation control via `create_vpn_gateway` flag
- Label support for all VPN components (gateway, tunnels, external gateway)
- Custom external VPN gateway descriptions
- Configurable IPsec shared secret length (default: 8 bytes)