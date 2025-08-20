
# 🧠 README – BGP Lab: Why `network` and `ip route` Failed

## ⚠️ Problem Summary

While configuring BGP in the lab, the following was attempted:

```bash
router bgp 65001
 network 66.66.66.0 mask 255.0.0.0
 network 67.66.66.0 mask 255.0.0.0
```

With supporting static routes:

```bash
ip route 66.66.66.0 255.0.0.0 Null0
ip route 67.66.66.0 255.0.0.0 Null0
```

Result:
```
%Inconsistent address and mask
```

---

## 🔍 Root Cause Analysis

### 1. Static Route Error (`ip route`)

Cisco IOS rejected the routes because the IP address did **not align with the subnet mask**.

- A `/8` mask (`255.0.0.0`) requires the **last three octets to be zero**.
- `66.66.66.0` and `67.66.66.0` violate that rule.

#### ✅ Valid examples:

```bash
ip route 66.0.0.0 255.0.0.0 Null0
ip route 67.0.0.0 255.0.0.0 Null0
```

### 2. BGP `network` Statement Rejection

Per **RFC 4271 – Section 9.1.2.2**:

> BGP will only advertise a route defined by the `network` command **if and only if that exact prefix exists in the routing table** (RIB).

So even if syntactically correct, your `network` statement will silently fail unless the exact prefix/mask combo exists.

---

## ✅ Final Fix Used

```bash
ip route 66.66.66.0 255.255.255.0 Null0
ip route 67.66.66.0 255.255.255.0 Null0

router bgp 65001
 network 66.66.66.0 mask 255.255.255.0
 network 67.66.66.0 mask 255.255.255.0
```

Now both the static routes and BGP network entries align perfectly.

---

## 📄 Spec References

| Standard     | Section    | Insight                                                    |
|--------------|------------|-------------------------------------------------------------|
| RFC 4271     | §9.1.2.2   | BGP requires RIB route match for `network` to take effect   |
| Cisco IOS    | —          | Rejects `ip route` when address & mask do not align properly|

---

## 🧠 Key Takeaways

- `network` in BGP ≠ route injection — it **requires an exact RIB match**
- `ip route` must follow subnetting logic strictly (host bits = 0)
- Use `Null0` routes to force injection when needed
- Always validate with `show ip route` before adding BGP `network` entries
