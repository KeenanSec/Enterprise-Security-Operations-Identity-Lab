## Problem

DNS was not resolving upstream. The firewall rule allowed Pi-hole to reach `WAN net` instead of `any`, so queries to public resolvers were dropped.

## Fix

Changed the rule Destination from `WAN net` to `any`.
