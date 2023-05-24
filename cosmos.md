# cosmos async 操作

> Pip install azure-cosmos
>
> Pip install aiohttp

```python
import asyncio
import os
from contextlib import suppress
from typing import Dict

from azure.cosmos import PartitionKey
from azure.cosmos.aio import CosmosClient, ContainerProxy
from azure.cosmos.exceptions import CosmosResourceNotFoundError


async def create_item(container_client: ContainerProxy, item: Dict):
    await container_client.create_item(item)


async def main():
    async with CosmosClient.from_connection_string(
            os.environ['CosmosDBConnectionString']
    ) as client:
        database = client.get_database_client("odcpersistence")
        with suppress(CosmosResourceNotFoundError):
            await database.delete_container('mappings')

        container_client = await database.create_container(
            id="mappings", partition_key=PartitionKey(path="/id")
        )

        jobs = [
            create_item(container_client, {"id": "id"})
        ]
        await asyncio.gather(*jobs)


if __name__ == '__main__':
    asyncio.run(main())

```
