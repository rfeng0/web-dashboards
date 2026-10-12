# Setting up Cloudflare for the dashboards

> **Who this is for:** the team member with access to the `sfuracerbot.ca` Cloudflare account, adding or removing a car or team member.
> **Read first:** [../README.md](../README.md) (the diagram), and [security.md](security.md) for why each lock exists.
> **You'll be able to:** give team members access to the dashboard, and add a car with a config change.
> **Time:** ten minutes per extra car.

Most of this is clicking in the Cloudflare dashboard, because it sets up who is allowed in, and that should not live in a repo. Steps marked **👤 Dashboard** can only be done by a person there. A few checks are commands in a terminal.

## Adding a car

For a car `rb3`:

1. Go to **Zero Trust** > **Access controls** > **Applications**, **Create new application**, **Self-hosted and private**.
2. Name it `rb3 origins`. Select **Add public hostname** three times and enter:
   - `rb3-dash-origin` . `sfuracerbot.ca`
   - `rb3-bridge-origin` . `sfuracerbot.ca`
   - `rb3-cam-origin` . `sfuracerbot.ca`
3. Under **Access policies**, create a policy named `worker only` with **Action: Service Auth**, and one **Include** rule: **Service Token** is `racerbot-dashboard-worker`. Add no other policy.
4. Under **Additional settings**, turn on **401 Response for Service Auth policies**, so a missing token gets a plain 401 rather than a login page.
5. Select **Create**.

6. Go to **Networking** > **Tunnels**, select the car's tunnel, open the **Routes** tab.
7. **Add route** > **Published application**, three times:

   | Subdomain | Domain | Service |
   |---|---|---|
   | `rb3-dash-origin` | `sfuracerbot.ca` | `http://localhost:8080` (dashboard_node) |
   | `rb3-bridge-origin` | `sfuracerbot.ca` | `http://localhost:8765` (foxglove_bridge) |
   | `rb3-cam-origin` | `sfuracerbot.ca` | `http://localhost:9090` (usb_cam_stream) |

8. **In the repo** — add one entry to `CARS` in [`wrangler.jsonc`](../wrangler.jsonc) and open a pull request:

   ```jsonc
   "rb3": {
     "name": "Car 3",
     "dash_origin": "https://rb3-dash-origin.sfuracerbot.ca",
     "bridge_origin": "https://rb3-bridge-origin.sfuracerbot.ca",
     "cam_origin": "https://rb3-cam-origin.sfuracerbot.ca"
   }
   ```

Merging deploys it. **Working when:** the landing page lists the new car. The same service token and the same email list cover it; no code changes.

## Adding a person

Add their email to the `team` policy in **Access Policies**


## Removing access

| To remove | Do |
|---|---|
| A person | Remove their email from the `team` policy in **Access Policies**. |
| The Worker's access to the cars | Revoke the service token **Zero Trust** > **Access controls** > **Service credentials** > **Service Tokens**. Every car origin then refuses the Worker. |
| A car | Delete its `CARS` entry and merge; then delete its tunnel routes and origin application. |
