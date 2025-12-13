# tfmodule-google-vpn-ha

Terraform module for creating and managing Google Cloud HA VPN gateways with BGP routing and multiple tunnel configurations.

## Features

- **High Availability VPN Gateway** - Creates HA VPN gateway with automatic failover support
- **External VPN Gateway** - Configures external VPN gateway for connecting to on-premises or third-party VPNs
- **Automatic Router Creation** - Optionally creates Cloud Router with BGP configuration
- **Multiple Tunnel Support** - Manages multiple VPN tunnels with individual BGP sessions
- **BGP Routing** - Full BGP configuration with custom advertisement and route priorities
- **Interconnect Integration** - Supports IPsec-encrypted Cloud Interconnect via VPN interfaces
- **Flexible Peering** - Supports both external gateway and GCP-to-GCP VPN connections
- **Automatic Secret Generation** - Generates secure shared secrets automatically if not provided
- **Custom Route Advertisement** - Configurable BGP advertisement modes and IP ranges
- **Policy-Based Routing** - Supports import and export policies for BGP sessions
- **Dual-Stack Support** - Configurable IP stack type (IPv4/IPv6)

## Usage

### Basic HA VPN to External Gateway

```hcl
module "vpn_ha" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/my-vpc"
  name       = "my-vpn-gateway"

  router_asn = 64515

  peer_external_gateway = {
    redundancy_type = "TWO_IPS_REDUNDANCY"
    interfaces = [
      {
        id         = 0
        ip_address = "8.8.8.8"
      },
      {
        id         = 1
        ip_address = "8.8.4.4"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 1
      vpn_gateway_interface           = 1
      ike_version                     = 2
    }
  }
}
```

### GCP-to-GCP HA VPN

```hcl
module "vpn_ha_gcp" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/vpc-a"
  name       = "vpn-to-vpc-b"

  router_asn = 64515

  peer_gcp_gateway = "projects/peer-project/regions/us-central1/vpnGateways/peer-gateway"

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range     = "169.254.1.2/30"
      vpn_gateway_interface = 0
      ike_version           = 2
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range     = "169.254.2.2/30"
      vpn_gateway_interface = 1
      ike_version           = 2
    }
  }
}
```

### With Existing Router

```hcl
module "vpn_ha_existing_router" {
  source = "./tfmodule-google-vpn-ha"

  project_id  = "my-project-id"
  region      = "us-central1"
  network     = "projects/my-project-id/global/networks/my-vpc"
  name        = "my-vpn-gateway"
  router_name = "my-existing-router"

  peer_external_gateway = {
    redundancy_type = "SINGLE_IP_INTERNALLY_REDUNDANT"
    interfaces = [
      {
        id         = 0
        ip_address = "203.0.113.1"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 1
      ike_version                     = 2
    }
  }
}
```

### With Custom BGP Advertisement

```hcl
module "vpn_ha_custom_bgp" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/my-vpc"
  name       = "my-vpn-gateway"

  router_asn = 64515

  router_advertise_config = {
    mode = "CUSTOM"
    groups = [
      "ALL_SUBNETS"
    ]
    ip_ranges = {
      "10.0.0.0/8"  = "Private network"
      "172.16.0.0/12" = "Additional range"
    }
  }

  peer_external_gateway = {
    redundancy_type = "TWO_IPS_REDUNDANCY"
    interfaces = [
      {
        id         = 0
        ip_address = "203.0.113.1"
      },
      {
        id         = 1
        ip_address = "203.0.113.2"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 1
      vpn_gateway_interface           = 1
      ike_version                     = 2
    }
  }
}
```

### With Per-Tunnel BGP Options and Route Policies

```hcl
module "vpn_ha_advanced_bgp" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/my-vpc"
  name       = "my-vpn-gateway"

  router_asn     = 64515
  route_priority = 1000

  peer_external_gateway = {
    redundancy_type = "TWO_IPS_REDUNDANCY"
    interfaces = [
      {
        id         = 0
        ip_address = "203.0.113.1"
      },
      {
        id         = 1
        ip_address = "203.0.113.2"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2

      bgp_peer_options = {
        route_priority   = 100
        advertise_mode   = "CUSTOM"
        advertise_groups = ["ALL_SUBNETS"]
        advertise_ip_ranges = {
          "192.168.0.0/16" = "Custom range for tunnel 0"
        }
        import_policies = [
          "projects/my-project-id/regions/us-central1/routePolicies/import-policy-1"
        ]
        export_policies = [
          "projects/my-project-id/regions/us-central1/routePolicies/export-policy-1"
        ]
      }
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 1
      vpn_gateway_interface           = 1
      ike_version                     = 2

      bgp_peer_options = {
        route_priority = 200
      }
    }
  }
}
```

### With Custom Shared Secrets

```hcl
module "vpn_ha_custom_secrets" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/my-vpc"
  name       = "my-vpn-gateway"

  router_asn = 64515

  peer_external_gateway = {
    redundancy_type = "TWO_IPS_REDUNDANCY"
    interfaces = [
      {
        id         = 0
        ip_address = "203.0.113.1"
      },
      {
        id         = 1
        ip_address = "203.0.113.2"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2
      shared_secret                   = "my-custom-secret-key-1"
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 1
      vpn_gateway_interface           = 1
      ike_version                     = 2
      shared_secret                   = "my-custom-secret-key-2"
    }
  }
}
```

### Using Existing VPN Gateway

```hcl
module "vpn_ha_existing_gateway" {
  source = "./tfmodule-google-vpn-ha"

  project_id            = "my-project-id"
  region                = "us-central1"
  network               = "projects/my-project-id/global/networks/my-vpc"
  name                  = "additional-tunnels"
  create_vpn_gateway    = false
  vpn_gateway_self_link = "projects/my-project-id/regions/us-central1/vpnGateways/existing-gateway"

  router_asn = 64515

  peer_gcp_gateway = "projects/peer-project/regions/us-central1/vpnGateways/peer-gateway"

  tunnels = {
    tunnel-2 = {
      bgp_peer = {
        address = "169.254.3.1"
        asn     = 64517
      }
      bgp_session_range     = "169.254.3.2/30"
      vpn_gateway_interface = 0
      ike_version           = 2
    }
  }
}
```

### With Interconnect Attachment (IPsec-encrypted Cloud Interconnect)

```hcl
module "vpn_ha_interconnect" {
  source = "./tfmodule-google-vpn-ha"

  project_id = "my-project-id"
  region     = "us-central1"
  network    = "projects/my-project-id/global/networks/my-vpc"
  name       = "vpn-interconnect-gateway"

  router_asn = 64515

  interconnect_attachment = [
    "projects/my-project-id/regions/us-central1/interconnectAttachments/attachment-1",
    "projects/my-project-id/regions/us-central1/interconnectAttachments/attachment-2"
  ]

  peer_external_gateway = {
    redundancy_type = "TWO_IPS_REDUNDANCY"
    interfaces = [
      {
        id         = 0
        ip_address = "203.0.113.1"
      },
      {
        id         = 1
        ip_address = "203.0.113.2"
      }
    ]
  }

  tunnels = {
    tunnel-0 = {
      bgp_peer = {
        address = "169.254.1.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.1.2/30"
      peer_external_gateway_interface = 0
      vpn_gateway_interface           = 0
      ike_version                     = 2
    }
    tunnel-1 = {
      bgp_peer = {
        address = "169.254.2.1"
        asn     = 64516
      }
      bgp_session_range               = "169.254.2.2/30"
      peer_external_gateway_interface = 1
      vpn_gateway_interface           = 1
      ike_version                     = 2
    }
  }
}
```

## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|------|---------|:--------:|
| name | VPN gateway name, and prefix used for dependent resources | `string` | n/a | yes |
| project_id | Project where resources will be created | `string` | n/a | yes |
| region | Region used for resources | `string` | n/a | yes |
| network | VPC used for the gateway and routes | `string` | n/a | yes |
| router_asn | Router ASN used for auto-created router | `number` | `64514` | no |
| router_name | Name of router, leave blank to create one | `string` | `""` | no |
| router_advertise_config | Router custom advertisement configuration, ip_ranges is a map of address ranges and descriptions | `object({groups=list(string), ip_ranges=map(string), mode=optional(string)})` | `null` | no |
| keepalive_interval | The interval in seconds between BGP keepalive messages that are sent to the peer | `number` | `20` | no |
| route_priority | Route priority, defaults to 1000 | `number` | `1000` | no |
| peer_external_gateway | Configuration of an external VPN gateway to which this VPN is connected | `object({name=optional(string), redundancy_type=optional(string), interfaces=list(object({id=number, ip_address=string}))})` | `null` | no |
| peer_gcp_gateway | Self Link URL of the peer side HA GCP VPN gateway to which this VPN tunnel is connected | `string` | `null` | no |
| tunnels | VPN tunnel configurations, bgp_peer_options is usually null | `map(object({...}))` | `{}` | no |
| create_vpn_gateway | Create a VPN gateway | `bool` | `true` | no |
| vpn_gateway_self_link | self_link of existing VPN gateway to be used for the vpn tunnel. create_vpn_gateway should be set to false | `string` | `null` | no |
| stack_type | The IP stack type will apply to all the tunnels associated with this VPN gateway | `string` | `"IPV4_ONLY"` | no |
| interconnect_attachment | URL of the interconnect attachment resource. When the value of this field is present, the VPN Gateway will be used for IPsec-encrypted Cloud Interconnect | `list(string)` | `[]` | no |
| labels | Labels for vpn components | `map(string)` | `{}` | no |
| external_vpn_gateway_description | An optional description of external VPN Gateway | `string` | `"Terraform managed external VPN gateway"` | no |
| ipsec_secret_length | The length of shared secret for VPN tunnels | `number` | `8` | no |

### Tunnel Object Structure

Each tunnel in the `tunnels` map supports the following attributes:

| Name | Description | Type | Required |
|------|-------------|------|:--------:|
| bgp_peer | BGP peer configuration with address and ASN | `object({address=string, asn=number})` | yes |
| bgp_session_name | Custom name for BGP session (defaults to `{name}-{tunnel_key}`) | `string` | no |
| bgp_session_range | BGP session IP range in CIDR notation | `string` | no |
| ike_version | IKE protocol version (1 or 2) | `number` | no |
| vpn_gateway_interface | VPN gateway interface ID (0 or 1) | `number` | no |
| peer_external_gateway_interface | Peer external gateway interface ID | `number` | no |
| peer_external_gateway_self_link | Override peer external gateway self link for this tunnel | `string` | no |
| shared_secret | Custom shared secret (auto-generated if empty) | `string` | no |
| bgp_peer_options | Advanced BGP options for the tunnel | `object({...})` | no |

### BGP Peer Options Structure

| Name | Description | Type |
|------|-------------|------|
| ip_address | BGP interface IP address | `string` |
| advertise_mode | BGP advertisement mode (DEFAULT or CUSTOM) | `string` |
| advertise_groups | List of advertised groups (e.g., ALL_SUBNETS) | `list(string)` |
| advertise_ip_ranges | Map of IP ranges to advertise with descriptions | `map(string)` |
| route_priority | Route priority override for this tunnel | `number` |
| import_policies | List of import policy self links | `list(string)` |
| export_policies | List of export policy self links | `list(string)` |

## Outputs

| Name | Description |
|------|-------------|
| gateway | HA VPN gateway resource |
| external_gateway | External VPN gateway resource |
| name | VPN gateway name |
| self_link | HA VPN gateway self link |
| router | Router resource (only if auto-created) |
| router_name | Router name |
| tunnels | VPN tunnel resources (sensitive) |
| tunnel_names | VPN tunnel names |
| tunnel_self_links | VPN tunnel self links (sensitive) |
| random_secret | Generated secret (sensitive) |

## Requirements

| Name | Version |
|------|---------|
| terraform | >= 1.5.0 |
| google | >= 7.0.0, < 8.0.0 |
| google-beta | >= 7.0.0, < 8.0.0 |

## Changelog

See [CHANGELOG.md](./CHANGELOG.md) for version history and changes.