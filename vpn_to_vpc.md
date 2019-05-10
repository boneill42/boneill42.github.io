
How to get secure access to your VPC

* Follow [these instructions](https://docs.aws.amazon.com/vpn/latest/clientvpn-admin/authentication-authrization.html#mutual) to generate your certificates.  Mark down your client and server certificate arns.

* Configure the client endpoint [here](https://console.aws.amazon.com/vpc/home?region=us-east-1#CreateClientVpnEndpoint:)

* You can use 0.0.0.0/0 to allow client connnections from anywhere.
