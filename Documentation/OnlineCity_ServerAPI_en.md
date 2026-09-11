### Server API
The server supports an HTTP API for **POST requests** in **JSON (UTF-8)** or `multipart/form-data` format.
Requests are sent to the same port as the game connection. A single TCP connection processes only one request and is then closed.

For C#, you can use `OnlineCityAPIClient.cs`. A complete list of commands is described below and in `OnlineCityAPIModel.cs`.

**Request example:**
```json
{ “Q”: “s” }
```

### API Commands
| `Q`          | Description                                                                                        | Specifications            |
| ------------ | -------------------------------------------------------------------------------------------------- | ------------------------- |
| `s`          | Server status: players online, total number of players, and a list of those awaiting confirmation. | `Key` — for `NeedApprove` |
| `p`          | Information about a specific player.                                                               | `Login`                   |
| `a`          | Information about all registered players.                                                          | —                         |
| `PlayerStat` | A complete report on the players, similar to the `S` command in the console.                       | `Key`                     |
| `BlockKey`   | Returns the contents `blockkey.txt`.                                                               | `Key`                     |
| `cc`         | Executes a single-letter server console command.                                                   | `N`, `Key`                |
| `cp`         | Sends a message to the general game chat.                                                          | `N`, `Key`                |
| `pa`         | Approves or rejects player registrations.                                                          | `Login`, `N`, `Key`       |
| `li`         | Retrieves an image from the server.                                                                | `N`, `Key`                |
| `si`         | Saves the image to the server.                                                                     | `N`, `Data`, `Key`        |

### Query Parameters
| Parameter | Description                                                                                  |
| -------- | --------------------------------------------------------------------------------------------- |
| `Q`      | API command code. Case-insensitive.                                                           |
| `Login`  | Player login. For `pa`, you can specify multiple logins using `*`.                            |
| `N`      | Additional command parameter: console command, message text, logins, or image name.           |
| `Key`    | Access key for protected commands. Must match the server’s `SecretKey`.                       |
| `Data`   | Image data for the `si` command.                                                              |
| `T`      | Additional numeric parameter. Its purpose is determined by the API handler.                   |

### Response `s`
Returns:
| Field          | Description                                                 |
| ------------- | ------------------------------------------------------------ |
| `OnlineCount` | Number of players online.                                    |
| `PlayerCount` | Total number of registered players.                          |
| `Onlines`     | List of usernames of players currently online.               |
| `NeedApprove` | Players awaiting approval. Format: `username*Discord`.       |

### Response `p` and `a`
Return a list of `Players` containing information about the players:
| Field              | Description                                 |
| ------------------ | ------------------------------------------- |
| `Login`            | Player's login.                             |
| `DiscordUserName`  | Discord username.                           |
| `LastOnlineTime`   | Last activity time in UTC.                  |
| `Days`             | Number of in-game days. `Days / 60` ≈ years.|
| `BaseCount`        | Number of settlements.                      |
| `CaravanCount`     | Number of caravans.                         |
| `MarketValueTotal` | Total market value of property.             |
| `BaseServerIds`    | Settlement IDs, separated by commas.        |

### Images
The `li` and `si` commands use `multipart/form-data`.
The `N` parameter specifies the image name:
* `pl_Nick` — the player's avatar.
* `cs_Nick@serverId` — a screenshot of the colony.

`serverId` can be obtained via `a` or `p` in the `BaseServerIds` field.
A ready-made example of `multipart/form-data` can be found in `Documentation\test.html`.

### Registration Approval
To manually approve players, enable:
* `PlayerNeedApprove`
* `PlayerNeedApproveInDiscord`

Currently, approval is available only through Discord.

The `s` command with the `Key` parameter returns a `NeedApprove` list in the following format:
```text
Nickname*Discord
```

Use `pa` for approval:
* `Login` — usernames of approved players separated by `*`;
* `N` — usernames of rejected players separated by `*`.

Rejected players are removed, after which their usernames can be registered again.

### Errors
In the event of an error, the API returns:
```json
{
  “Error”: “Error description”
}
```

### Testing via Telnet
You can use `telnet` to quickly test the API:
```text
o 127.0.0.1 19019

POST / HTTP/1.1

{Q:“s”}
```

Example response:
```text
HTTP/1.0 200 OK
Content-Type: application/json; charset=utf-8
Connection: close

{“OnlineCount”:0,‘PlayerCount’:2,“Onlines”:[]}
```
