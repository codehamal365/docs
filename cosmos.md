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



# cosmos crud

```python
from azure.cosmos import CosmosClient, ContainerProxy


class ContainerCrud:
    def __init__(self, proxy: ContainerProxy):
        self.proxy = proxy

    def query_items(
            self, query: str, parameters: dict = None, handler=lambda items: items
    ) -> any:
        return handler(
            self.proxy.query_items(
                query=query,
                parameters=[
                    {"name": key, "value": value} for key, value in parameters.items()
                ]
                if parameters
                else [],
                enable_cross_partition_query=True,
            )
        )

    def read_all_items(
            self, handler=lambda items: items
    ) -> any:
        return handler(
            self.proxy.read_all_items()
        )

    def delete_item(self, item_id: str):
        self.proxy.delete_item(
            item=item_id,
            partition_key=item_id,
        )

    def create_item(self, data: dict) -> dict:
        return self.proxy.create_item(data)


class Cosmos:
    def __init__(self, database, container, connection):
        self.connection = connection
        self.database = database
        self.container = container
        self.client = None
        self.container_crud = None

    def __enter__(self):
        self.client = CosmosClient.from_connection_string(self.connection)
        self.container_crud = ContainerCrud(
            self.client.get_database_client(self.database).get_container_client(
                self.container
            )
        )
        return self

    def __exit__(self, *args):
        return self.client.__exit__(*args)
      
  
  if __name__ == '__main__':
    with Cosmos(database='database', container='container', connection='connection') as client:
        print(client.container_crud.read_all_items())
```

