### Using Service Locator pattern in SPFx with Library components
 
Create a custom service which exposes operations with MSGraphClient and host it in a SPFx library component. Consume this service from a standard SPFx web part.

Custom service hosted in an SPFx library component:

```ts
import { ServiceKey, ServiceScope } from '@microsoft/sp-core-library';
import { MSGraphClientFactory, MSGraphClient } from '@microsoft/sp-http';

export interface ICustomGraphService {
    getMyDetails(): Promise<JSON>;
}

export class CustomGraphService implements ICustomGraphService {
    
    //Create a ServiceKey which will be used to consume the service.
    public static readonly serviceKey: ServiceKey<ICustomGraphService> =
        ServiceKey.create<ICustomGraphService>('my-custom-app:ICustomGraphService', CustomGraphService);

    private _msGraphClientFactory: MSGraphClientFactory;

    constructor(serviceScope: ServiceScope) {
        serviceScope.whenFinished(() => {
            this._msGraphClientFactory = serviceScope.consume(MSGraphClientFactory.serviceKey);
        });
    }

    public getMyDetails(): Promise<JSON> {
        return new Promise<JSON>((resolve, reject) => {
            this._msGraphClientFactory.getClient().then((_msGraphClient: MSGraphClient) => {
                _msGraphClient.api('/me').get((error, user: JSON, rawResponse?: any) => {
                    resolve(user);
                });
            });
        });
    }
}
```

Consuming the service from an SPFx webpart or an extension:

```js
//package.json
"dependencies": {
    "corporate-library": "0.0.1"
    //other dependencies 
}
```

```ts
import { CustomGraphService } from 'corporate-library';

const graphServiceInstance = this.context.serviceScope.consume(CustomGraphService.serviceKey);

graphServiceInstance.getMyDetails().then((user: JSON) => {
    console.log(user);
});
```

More details about SPFx library components: https://docs.microsoft.com/en-us/sharepoint/dev/spfx/library-component-tutorial

