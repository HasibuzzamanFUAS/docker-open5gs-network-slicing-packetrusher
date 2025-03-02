# Overview of the open5gs network slicing deployment

The Open5GS `network-slicing` deployment is designed to work with PacketRusher, which simulates gNBs and UEs, while only exposing the `MongoDB database` via TCP port `27017`.

This deployment integrates `PacketRusher` (gNB and UE) without building Docker images from this repository. Instead, it pulls pre-built images from the `ghcr.io/borjis131` registry, which are originally built from this repository.

The setup connects three UEs (`UE1, UE2, and UE3`) to three gNBs, utilizing three separate PacketRusher files. This approach is necessary due to PacketRusher’s known limitation of supporting `multi-ue` vrf tunneling within a Docker network [here](https://github.com/Borjis131/docker-open5gs/issues/15#issuecomment-2669900076). The UEs are configured to select three different network slices based on their `S-NSSAI`, all of which are defined in the Open5GS 5G Core configuration files.

UE1 configuration:
```yaml
dnn: internet
sst: 1
sd: 000001
```
UE2 configuration:
```yaml
dnn: internet
sst: 2
sd: 000001
```
UE3 configuration: 
```yaml
dnn: internet
sst: 3
sd: 000001
```

Upon receiving this configuration in the 5G Core, the dedicated AMF uses it to select the appropriate SMF.

for SMF1:
```yaml
- plmn_id:
  mcc: 262
  mnc: 02
s_nssai:
  - sst: 1
    sd: 000001
```
and so on.


SMF1 is configured to serve the `sst: 1 and sd: 000001`
SMF1 is connected to UPF1, serving IP addresses at the DNN internet from the pool `10.45.0.0/16` 
SMF2 is configured to serve the `sst: 2 and sd: 000001`. 
SMF2 is connected to UPF2, serving IP addresses at the DNN internet from the pool `10.46.0.0/16`
SMF3 is configured to serve the `sst: 3 and sd: 000001`
SMF3 is connected to UPF3, serving IP addresses at the DNN internet from the pool `10.47.0.0/16` 


# Successful operation

When testing this, after adding the UEs through `webui` portal(`user: admin pass: 1423`) to the database with the appropriate values, UE1 should receive IP address `10.45.0.2` and be assigned to Slice 1, UE2 should receive IP address `10.46.0.2` and be assigned to Slice 2 and UE3 should receive IP address `10.47.0.2` and be assigned to Slice 3.
Each UE will successfully select its respective slice and operate as expected within the defined network slicing framework


[FAQs](https://github.com/Borjis131/docker-open5gs/blob/main/docs/FAQ.md), [Configurations](https://github.com/Borjis131/docker-open5gs/blob/main/docs/configuration.md)
