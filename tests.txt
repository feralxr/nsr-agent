1. From a lab PC, can you even reach the server?

Test-NetConnection <server-ip> -Port 8080
TcpTestSucceeded : False → firewall / network problem (most likely).
True → it's an agent-side issue, jump to step 4.
2. Open the firewall on the server PC (this is in your server/README.md but easy to miss):

New-NetFirewallRule -DisplayName "Lab Monitor" -Direction Inbound -Protocol TCP -LocalPort 8080 -Action Allow -Profile Private,Domain
3. Check the server PC's network profile. If Windows classifies the LAN as Public, the rule above (Private/Domain) won't apply:

Get-NetConnectionProfile
Either change it to Private, or add -Profile Public to the firewall rule.

4. Check the agent actually started and read the right config. The agent is a Windows service — if config.json is malformed or the service didn't start, there are no logs on the server. On the lab PC:

Get-Service LabAgent   # or whatever name you installed it under
Get-EventLog -LogName Application -Source "lab-agent" -Newest 20
5. Confirm the URL the agent is dialing. From your config.rs it builds ws://<server>:<port>/agent. Open a browser on the lab PC and hit http://<server-ip>:8080/ — if the viewer UI loads, the agent should be able to connect too.

Quick sanity question: are the lab PCs and the server on the same subnet/VLAN? Some school/lab networks isolate client ports from each other ("AP isolation" on Wi-Fi, or switch port isolation), which would let the server serve the viewer to your laptop but block PC-to-PC traffic entirely.

Want me to add a small "connection self-test" mode to the agent (log the resolved URL + a TCP probe result to a local file before opening the WebSocket) so you can diagnose this on lab PCs without a console?

Lovable update
