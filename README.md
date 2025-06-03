# Valve A2S Glance Widget

A [Glance](https://github.com/glanceapp/glance) widget to show information about game servers using Valve's A2S protocol. Requires an API middleman container. 

<img src=example.png width="400">

The thumbnail changes based on the map being played. Also shows a player list while hovering the player count.

Supported Games: Half-Life 2, Half-Life, Team Fortress 2, Counter-Strike: Global Offensive, Counter-Strike 1.6, ARK: Survival Evolved, Rust

### API Setup

See [A2S-API](https://github.com/erkston/a2s-api). It's extremely simple but required for the widget.

### Widget yaml

<details>

<summary>Click</summary>

```yaml
- type: custom-api
  title: Team Fortress 2
  cache: 30s
  options:
    server-ip: "chi-1.us.uncletopia.com"
    server-port: 27015
    nice-name: "Uncletopia Chicago 1"
    api-url: "192.168.1.24:27014"
    #thumb-host: "https://my.host/fastdl/map-images/"
  template: |
    {{ $serverIp := .Options.StringOr "server-ip" "127.0.0.1" }}
    {{ $serverPort := .Options.IntOr "server-port" 27015 }}
    {{ $apiUrl := .Options.StringOr "api-url" "" }}
    {{ $queryUrl := printf "http://%s/query?ip=%s&port=%d" $apiUrl $serverIp $serverPort }}

    {{ $request := newRequest $queryUrl }}
    {{ $response := $request | getResponse }}
    {{ $info := $response.JSON }}

    {{ $serverInfo := $info.Get "server_info" }}

    {{ $niceName := .Options.StringOr "nice-name" $serverIp }}
    {{ $thumbHost := .Options.StringOr "thumb-host" "https://raw.githubusercontent.com/erkston/valve-a2s-glance-widget/refs/heads/main/map-images/" }}
    {{ $thumbUrl := print $thumbHost ($serverInfo.String "map_name") ".png" }}

    <div style="display:flex; align-items:center; gap:12px;">
      <!-- Thumbnail -->
      <div style="width:64px; height:48px; overflow:hidden; border-radius:4px; flex-shrink:0;">
        <img src="{{ $thumbUrl }}" style="object-fit:cover;" alt="{{ $serverInfo.String "map_name" }}">
      </div>

      <!-- Server Info -->
      <div style="flex-grow:1; min-width:0;">
        <a class="size-h4 block text-truncate color-highlight">
          <span style="cursor: help;" data-popover-type="text" data-popover-text="{{ $serverInfo.String "server_name" }}">{{ $niceName }}</span>
          {{ if $serverInfo.String "server_name" }}
          <span
            style="width: 8px; height: 8px; border-radius: 50%; background-color: var(--color-positive); display: inline-block; vertical-align: middle;"
            data-popover-type="text" data-popover-text="Online"></span>
          {{ else }}
          <span
            style="width: 8px; height: 8px; border-radius: 50%; background-color: var(--color-negative); display: inline-block; vertical-align: middle;"
            data-popover-type="text" data-popover-text="Offline"></span>
          {{ end }}
        </a>
        <ul style="display: flex; flex-direction: column; margin-top: 0.25rem">
          <li style="display: flex; align-items: center;">
            <div style="width: 1.25em; flex-shrink: 0; margin-right: 0.4em; display: flex; align-items: center;">
              <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24" style="height: 1.25em;">
                <path fill-rule="evenodd" d="M12 2.25c-3.728 0-6.75 3.022-6.75 6.75 0 4.342 6.075 11.005 6.322 11.268a.75.75 0 0 0 1.056 0c.247-.263 6.322-6.926 6.322-11.268 0-3.728-3.022-6.75-6.75-6.75ZM9.75 9a2.25 2.25 0 1 1 4.5 0a2.25 2.25 0 0 1-4.5 0Z" clip-rule="evenodd"/>
              </svg>
            </div>
            <div style="min-width: 0; line-height: 1.25; vertical-align: middle;" class="text-truncate">
              {{ $serverInfo.String "map_name" }}
            </div>
          </li>
          <li style="display: flex; align-items: center; margin-top: 0.5rem">
            <div style="width: 1.25em; flex-shrink: 0; margin-right: 0.4em; display: flex; align-items: center;">
              <svg xmlns="http://www.w3.org/2000/svg" fill="currentColor" viewBox="0 0 24 24" style="height: 1.25em;">
                <path fill-rule="evenodd" d="M7.5 6a4.5 4.5 0 1 1 9 0 4.5 4.5 0 0 1-9 0ZM3.751 20.105a8.25 8.25 0 0 1 16.498 0 .75.75 0 0 1-.437.695A18.683 18.683 0 0 1 12 22.5c-2.786 0-5.433-.608-7.812-1.7a.75.75 0 0 1-.437-.695Z" clip-rule="evenodd" />
              </svg>
            </div>
            <div style="min-width: 0; line-height: 1.25; vertical-align: middle; cursor: help;" data-popover-type="text"
                 data-popover-text="{{ range $index, $player := $info.Array "player_list" }}{{ if $index }} • {{ end }}{{ $player.String "name" }}{{ end }}">
              {{ $serverInfo.Int "player_count" }}/{{ $serverInfo.Int "max_players" }} players
            </div>
          </li>
        </ul>
      </div>
    </div>

```
</details>

### User variables/options

| Options           | Short Description                                       |
| ----------------- | ------------------------------------------------------- |
| server-ip         | IP or hostname of the server you would like to monitor  |
| server-port       | Port of the server you would like to monitor            |
| nice-name         | (optional) overrides the first line of the widget. defaults to ```server-ip``` |
| api-url           | IP or hostname of the A2S-API container                 |
| thumb-host        | (optional) If set will use this as the base url for the map thumbnails |

> [!TIP]  
> `thumb-host` - This repo has some thumbnails for TF2 maps in the [map-images](https://github.com/erkston/valve-a2s-glance-widget/tree/main/map-images) folder. Sourced from PrivateTwinkleToes at [Spriters-Resource](https://www.spriters-resource.com/pc_computer/tf2/sheet/71302/). As this is not comprehensive for the possible maps/games supported, you can also set your own ```thumb-host``` to provide your own image host. This option should point to a folder with ```.png``` files for each map name. Alternatively submit a PR/Issue to add other game/map images to this repo.

## Inspo
[not-first](https://github.com/not-first) - [Minecraft Glance widget](https://github.com/glanceapp/community-widgets/blob/main/widgets/minecraft-server-by-not-first/README.md)

[PR1NT3R](https://github.com/PR1NT3R) - [Satisfactory Glance widget](https://github.com/PR1NT3R/satisfactory-glance-widget/)