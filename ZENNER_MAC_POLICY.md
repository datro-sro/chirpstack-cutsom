# ZENNER MAC-command policy

This v4.17.0-based build supports disabling LoRaWAN MAC commands for selected
device profiles without disabling confirmed-uplink ACKs, application downlinks
or join-accepts.

## Device-profile tag

Add this tag to each vulnerable device profile:

```text
mac_commands_disabled=true
```

The value comparison is ASCII case-insensitive. Missing tags and values other
than `true` leave the profile's MAC behavior unchanged.

When enabled, the policy:

- ignores uplink MAC requests and answers without producing a MAC response;
- clears MAC responses prepared before downlink assembly;
- skips network-initiated MAC-command generation;
- leaves ACK, application queue and join behavior unchanged.

The global `[network] mac_commands_disabled` setting remains authoritative and
continues to disable MAC commands for every profile.

## Safe rollout

1. Keep global MAC commands disabled.
2. Apply the tag to every ZENNER device profile.
3. Deploy this image to the test cluster.
4. Confirm tagged devices receive ACK-only downlinks but no MAC commands.
5. Confirm an untagged sacrificial profile retains normal MAC behavior.
6. Only then consider enabling MAC commands globally.

New ZENNER profiles must receive the tag before devices are assigned to them.

## Build

Build the PostgreSQL image for the production cluster architecture:

```bash
docker buildx build \
  --platform linux/amd64 \
  --file Dockerfile.custom \
  --tag registry.digitalocean.com/poseidon-repo/chirpstack:4.17.0-zenner-mac-policy.1 \
  --load \
  .
```
