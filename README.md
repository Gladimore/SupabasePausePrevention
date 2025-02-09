# Supabase Pause Prevention ⏸️

![GitHub stars](https://img.shields.io/github/stars/travisvn/supabase-pause-prevention?style=social)
![GitHub forks](https://img.shields.io/github/forks/travisvn/supabase-pause-prevention?style=social)
![GitHub repo size](https://img.shields.io/github/repo-size/travisvn/supabase-pause-prevention)
![GitHub language count](https://img.shields.io/github/languages/count/travisvn/supabase-pause-prevention)
![GitHub top language](https://img.shields.io/github/languages/top/travisvn/supabase-pause-prevention)
![GitHub last commit](https://img.shields.io/github/last-commit/travisvn/supabase-pause-prevention?color=red)
![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Ftravisvn%2Fsupabase-pause-prevention&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)
[![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://img.shields.io/github/sponsors/travisvn)

🛑 Stop Supabase projects from pausing due to inactivity! :raised_hands:

> On the free-tier plan, projects that are inactive for more than 7 days are paused.


# Please star ⭐️ if you find this useful 


> [!TIP]
> __NEW & IMPROVED VERSION__ written in Python available at [supabase-inactive-fix repo](https://github.com/travisvn/supabase-inactive-fix)



## Solution / How it works

- Creating a _cron job_ (scheduled task) that makes a simple database call
  - _(This keeps your Supabase project active)_
- Fetching `keep-alive.ts` API endpoints for the other projects, as Vercel limits free-tier users to one cron job.

## Getting Started

**Super simple to add to your existing Supabase project!**

### Configure your main project

_Only 3 files matter_

- [`/app/api/keep-alive/route.ts`](app/api/keep-alive/route.ts) - API endpoint the cron job will execute
- [`/app/api/keep-alive/helper.ts`](app/api/keep-alive/helper.ts) - Helper functions called from `route.ts`
- [`/config/keep-alive-config.ts`](config/keep-alive-config.ts) - Configuration for your setup
- [`/vercel.json`](vercel.json) - Directs Vercel to periodically run the `keep-alive` endpoint

`utils/supabase` folder contains files provided in the Supabase docs for the [Next.js Web App demo — Supabase](https://supabase.com/docs/guides/getting-started/tutorials/with-nextjs)

Everything else is boilerplate from Next.js `create-next-app`

### Configuring your other Supabase projects

After selecting your primary project _(the one that implements the code provided in this repository)_, you'll want to add an API endpoint to your other Supabase projects

The only requirement is that this endpoint is reachable and makes a call to your Supabase database


> [!NOTE]
> API endpoint must make database call   
> Ensure the server doesn't cache this


<details>

<summary>Example of a setup using Prisma as an ORM</summary>

`/pages/api/keep-alive.ts` 

```typescript
// Next.js API route support: https://nextjs.org/docs/api-routes/introduction
import type { NextApiRequest, NextApiResponse } from 'next'
import { prisma } from 'src/server/db'

// See next example for contents of @/utils/helper
import { generateRandomString } from '@/utils/helper'

export default async function handler(
  _req: NextApiRequest,
  res: NextApiResponse
) {
  try {
    const randomString = generateRandomString()
    const dbResponse = await prisma.keepAlive.findMany({
      where: {
        name: {
          equals: randomString,
        }
      }
    })
    const successMessage = (dbResponse != null) ? `Success - found ${dbResponse.length} entries` : "Fail"
    res.status(200).json(successMessage)
  } catch (e) {
    res.status(401).send("There was an error")
  }
}
```

`/prisma/schema.prisma`

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["postgresqlExtensions"]
}

datasource db {
  provider   = "postgresql"
  url        = env("DATABASE_URL")
  extensions = [uuidOssp(map: "uuid-ossp")]
}

model KeepAlive {
  id     BigInt  @id @default(autoincrement())
  name   String? @default("")
  random String? @default(dbgenerated("gen_random_uuid()")) @db.Uuid
}
```
</details>

<details>

  <summary>Helper function for '/pages' directory (legacy support)</summary>

  `/utils/helper.ts`

  ```typescript
  const defaultRandomStringLength: number = 12
  
  const alphabetOffset: number = 'a'.charCodeAt(0)
  export const generateRandomString = (length: number = defaultRandomStringLength) => {
    let newString = ''
  
    for (let i = 0; i < length; i++) {
      newString += String.fromCharCode(alphabetOffset + Math.floor(Math.random() * 26))
    }
  
    return newString
  }
  ```
</details>

### Sample SQL 

Any table and column can be called, but if you'd rather go with a generic, here's a SQL query for a `keep-alive` table 

```sql
CREATE TABLE "keep-alive" (
  id BIGINT generated BY DEFAULT AS IDENTITY,
  name text NULL DEFAULT '':: text,
  random uuid NULL DEFAULT gen_random_uuid (),
  CONSTRAINT "keep-alive_pkey" PRIMARY key (id)
);

INSERT INTO
  "keep-alive"(name)
VALUES
  ('placeholder'),
  ('example');
```

> [!IMPORTANT]
> It is now **strongly recommended** to use a `keep-alive` table like the one above.   
> This is in light of the added features for _optional_ database insertion & deletion actions

___

### Sample response

Visiting `https://your-project-domain.vercel.app/api/keep-alive` 

```
Results for retrieving
'mzmgylpviofc' from 'keep-alive' at column 'name': []

Other Endpoint Results:
https://your-other-vercel-project-urls.vercel.app/api/keep-alive - Passed
https://your-other-supabase-app.com/api/keep-alive - Passed

```

<details>
<summary>Extended response (with insertion / deletion)</summary>
  
```
Results for retrieving entries from 'keep-alive' - 'name column: [{"name":"placeholder"},{"name":"random"}, ... ,{"name":"uujyzdnsbrgi"}]

Results for deleting
'uujyzdnsbrgi' from 'keep-alive' at column 'name': success
```

</details>

___

This is a [Next.js](https://nextjs.org/) project bootstrapped with [`create-next-app`](https://github.com/vercel/next.js/tree/canary/packages/create-next-app).
