---
title: Using the Web API
parent: Get Started
nav_order: 3
---

# Using the Web API
{: .no_toc }

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Raid Toolkit provides a web-accessible API that can be used both from the browser as well as from various other scripting languages. 
By default it is available on the local loopback interface on port 9090: `wss://localhost:9090`.

Any applications accessing RTK must run on the same computer as RTK. Cross-network access is not a supported scenario.

# .NET 6.0

1. Install the latest Raid.Client package from nuget:

    ```sh
    dotnet add package Raid.Client
    ```

2. Use the `Raid.Client.RaidToolkitClient` class to connect to the RTK process:

    ```cs
    RaidToolkitClient client = new();
    try
    {
        // connect to the RTK process
        client.Connect();
        // fetch the known user account information
        var accounts = await client.AccountApi.GetAccounts();
        // dump the first account in RaidExtractor format
        AccountDump dump = await client.AccountApi.GetAccountDump(accounts[0].Id);
        // ...
    }
    catch (Exception ex)
    {
        Console.WriteLine($"An error occurred: {ex.Message}");
    }
    finally
    {
        // make sure to disconnect when you're done!
        client.Disconnect();
    }
    ```

# TypeScript/JavaScript

1. Install `Raid.Client` nuget package:

    **NPM:**

    ```sh
    npm install @raid-toolkit/webclient
    ```

    **Yarn v1:**

    ```sh
    yarn add @raid-toolkit/webclient
    ```

2. Use the `useRaidToolkitApi` method to construct the API you wish to use:

    ```ts
    import { useRaidToolkitApi, IAccountApi } from '@raid-toolkit/webclient';

    const api = useRaidToolkitApi(IAccountApi);
    const account = (await api.getAccounts())[0];
    let accountDump = await api.getAccountDump(account.id);
    ```

# Python3

1. Install the `raidtoolkit` package:

    ```sh
    pip3 install raidtoolkit
    ```

2. Use the `RaidToolkitClient` class to connect to the RTK process:

    ```py
    import asyncio
    from raidtoolkit import RaidToolkitClient

    async def main():
        client = RaidToolkitClient()
        client.connect()
        accounts = await client.AccountApi.get_accounts()
        print(accounts)
        account = await client.AccountApi.get_account_dump(accounts[0]["id"])
        print(account)
        client.close()

    if __name__ == "__main__":
        asyncio.run(main())
    ```
