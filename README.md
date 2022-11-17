<!-- PROJECT SHIELDS -->

[![Quality Gate Status](https://sonarcloud.io/api/project_badges/measure?project=frango9000_hal-form-client&metric=alert_status)](https://sonarcloud.io/summary/new_code?id=frango9000_hal-form-client)
![Development](https://github.com/frango9000/hal-form-client/actions/workflows/verify.yml/badge.svg)
[![codecov](https://codecov.io/gh/frango9000/hal-form-client/branch/master/graph/badge.svg?token=CRJO7hUJFn)](https://codecov.io/gh/frango9000/hal-form-client)

# Hal Form Client

An **Angular** HTTP client to consume [HAL](https://webconcepts.info/specs/IETF/I-D/kelly-json-hal)
or [HAL-FORMS](https://rwcbook.github.io/hal-forms/) resources
<br />
[Report Bug · Request Feature](https://github.com/frango9000/hal-form-client/issues)

<!-- ABOUT THE PROJECT -->

## About The Project ⌨️

### Description 🛠️

This node package provides an **Angular** client that can easily access any backend that
serves [HAL](https://webconcepts.info/specs/IETF/I-D/kelly-json-hal)
or [HAL-FORMS](https://rwcbook.github.io/hal-forms/) resources. Apart from the client itself, it provides Resource, Link
and Template classes which will automatically be returned if your servers response is a valid `application/hal+json`
or `application/prs.hal-forms+json` . Hal Form Client works in Node.js and in the browser and only supports json APIs.

<!-- GETTING STARTED -->

## Getting Started 🚀

### Installation 🔧

```sh
npm install @frango9000/hal-form-client
```

<!-- USAGE EXAMPLES -->

## Usage 🏹

### Hal Form Service

In your angular application module import `HalFormClientModule.forRoot()` indicating the url to the root resource of your API

```typescript
@NgModule({
  declarations: [AppComponent],
  imports: [
    BrowserModule,
    BrowserAnimationsModule,
    AppRoutingModule,
    HttpClientModule,
    HalFormClientModule.forRoot('/api'),
  ],
  bootstrap: [AppComponent],
})
export class AppModule {}
```

Then you can inject the service using Angular DI initialize it.

```typescript
import { HalFormService } from '@frango9000/hal-form-client';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.scss'],
})
export class AppComponent implements OnInit {
  constructor(private halFormService: HalFormService) {}

  ngOnInit(): void {
    this.halFormService.initialize().subscribe((rootResource) => console.log(rootResource));
  }

  getRootLink(): Observable<Link> {
    this.halFormService.getLink().subscribe((link) => console.log(link));
  }
}
```

### Resource

HAL-FORM Resource example

```json
{
  "firstName": "John",
  "lastName": "Doe",
  "_links": {
    "self": {
      "href": "/api/users/1"
    },
    "update": {
      "href": "/api/users/1",
      "method": "PATCH",
      "title": "Update User",
      "contentType": "application/json",
      "properties": [
        {
          "name": "firstName",
          "type": "string",
          "required": true,
          "readOnly": false
        }
      ]
    }
  }
}
```

```typescript
import { Resource } from '@frango9000/hal-form-client';

const resource = new Resource({
  firstName: 'John',
  lastName: 'Doe',
  _links: {
    self: {
      href: '/api/users/1',
    },
  },
  _templates: {
    update: {
      method: 'PATCH',
      target: '/api/users/1',
      title: 'Update User',
      contentType: 'application/json',
      properties: [
        {
          name: 'firstName',
          type: 'string',
          required: true,
          readOnly: false,
        },
      ],
    },
  },
});

resource
  .getTemplate('update')
  .afford()
  .subscribe((resource) => {
    console.log(resource);
  });

Resource.of({ _links: { self: { href: '/api/users/1' } } })
  .getLink('self')
  .follow()
  .subscribe((resource) => {
    console.log(resource);
  });
```

###### Resource API:

- `getLink(): Link | null` - Returns the link if it exists

- `hasLink(): boolean` - Returns true if the link exists

- `follow<T>(): Observable<T extends Resource>` - Shortcut for resource.getLink().follow()

- `getEmbedded<T>(): Observable<T | T[]>` - Returns the nested embedded objects

- `hasEmbedded(): boolean` - Returns true if the nested embedded objects exist

- `hasEmbeddedCollection(): boolean` - Returns true if the nested embedded is a collection

- `getEmbeddedCollection<T>(): Observable<T[]>` - Returns the nested embedded is collection

- `hasEmbeddedObject(): boolean` - Returns true if the nested embedded is an object

- `getEmbeddedObject<T>(): Observable<T>` - Returns the nested embedded object

- `getTemplate(): Template | null` - Returns the template if it exists

- `hasTemplate(): boolean` - returns true if the template exists

- `affordTemplate<T>(): Observable<T>` - shortcut for resource.getTemplate().afford()

- `canAfford(): boolean` - shortcut for resource.getTemplate().canAfford()

- `canAffordProperty(): boolean` - shortcut for resource.getTemplate().canAffordProperty()

### Link

HAL Link example

```json
{
  "href": "/api/users/{userId}",
  "title": "User",
  "templated": true
}
```

```typescript
import { Link } from '@frango9000/hal-form-client';

const link = new Link({
  href: '/api/users/{userId}',
  title: 'User',
  templated: true,
});

link.follow().subscribe((resource) => {
  console.log(resource);
});

Link.ofHref('/api/users')
  .follow()
  .subscribe((resource) => {
    console.log(resource);
  });
```

###### Link API:

- `follow<T>(): Observable<T extends Resource>` - Follows the link and returns a new Resource initialized with the
  response body

- `followRaw<T>(): Observable<T>` - Follows the link and returns the json body response

- `get<T>(): Observable<HttpResponse<T>>` - Follows the link and returns the Http Response

### Template

HAL-FORM Template example

```json
{
  "method": "PATCH",
  "target": "/api/v1/users/1",
  "title": "Update User",
  "contentType": "application/json",
  "properties": [
    {
      "name": "firstName",
      "type": "string",
      "required": true,
      "readOnly": false
    }
  ]
}
```

```typescript
import { Template } from '@frango9000/hal-form-client';

const template = new Template({
  method: 'PATCH',
  target: '/api/v1/users/1',
  title: 'Update User',
  contentType: 'application/json',
  properties: [
    {
      name: 'firstName',
      type: 'string',
      required: true,
      readOnly: false,
    },
  ],
});

template.afford().subscribe((resource) => {
  console.log(resource);
});

Template.of({ method: 'PATCH', target: '/api/users' })
  .afford({ body: { firstName: 'new name' } })
  .subscribe((resource) => {
    console.log(resource);
  });
```

###### Template API:

- `afford<T>(): Observable<T extends Resource>` - Affords the template and returns a new Resource initialized with the
  response body

- `affordRaw<T>(): Observable<T>` - Affords the template and returns the json body response

- `request<T>(): Observable<HttpResponse<T>>` - Affords the template and returns the Http Response

- `canAfford(): boolean` - returns true if the input can be afforded by the template

- `canAffordProperty(): boolean` - returns true if the input property can be afforded by the template

<!-- ROADMAP -->

## Roadmap

See the [open issues](https://github.com/frango9000/hal-form-client/issues) for a list of proposed features (and known issues).

### Release 1.0.0

- [x] Core Implementation
- [x] Core Implementation Tests
- [ ] Documentation
- [ ] Examples
- [ ] Publish to NPM

#### Goals

Provide a free open source library to help developers to consume HAL/HAL-FORMS APIs.

<!-- LICENSE -->

## License 📄

## Wiki 📖

More info on the project can be found in
our [Wiki](https://github.com/frango9000/hal-form-client/wiki)

<!-- CONTRIBUTING -->

## Contributing 🖇️

Contributions are what make the open source community such an amazing place to learn, inspire,
and create. Any contributions you make are **greatly appreciated**.

1. Fork the Project
2. Create your Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit your Changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the Branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

<!-- BUILT -->

### Built With 🛠️

- [Typescript](https://www.typescriptlang.org/)
- [Node.js](http://nodejs.org/)
- [Yarn](https://yarnpkg.com/)
- [Angular](https://angular.io/)
- [Nx](https://nx.dev/angular)
- [IntelliJ IDEA](https://www.jetbrains.com/idea/)

<!-- CONTACT -->

## Contact ✒️

- **[Francisco Sanchez](https://frango9000.github.io/)**

[![Email][email-contact-shield]][email-contact-url]
[![Github][github-contact-shield]][github-contact-url]
[![LinkedIn][linkedin-contact-shield]][linkedin-contact-url]

<!-- SHARE -->

## Share 🔗

[![Email][email-share-shield]][email-share-url]
[![LinkedIn][linkedin-share-shield]][linkedin-share-url]
[![Facebook][facebook-share-shield]][facebook-share-url]
[![Twitter][twitter-share-shield]][twitter-share-url]

<!-- ACKNOWLEDGEMENTS -->

## Acknowledgements 🎁

<!-- MARKDOWN LINKS & IMAGES -->
<!-- https://www.markdownguide.org/basic-syntax/#reference-style-links -->

[linkedin-contact-shield]: https://img.shields.io/badge/-LinkedIn-black?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-contact-url]: https://www.linkedin.com/in/fsancheztemprano/
[github-contact-shield]: https://img.shields.io/badge/-Github-black?style=for-the-badge&logo=github&colorB=555
[github-contact-url]: https://github.com/frango9000
[email-contact-shield]: https://img.shields.io/badge/-email-black.svg?style=for-the-badge&colorB=555
[email-contact-url]: mailto:frango9000@gmail.com
[linkedin-share-shield]: https://img.shields.io/badge/Share-Linkedin?style=for-the-badge&logo=linkedin&colorB=555
[linkedin-share-url]: https://www.linkedin.com/in/fsancheztemprano
[facebook-share-shield]: https://img.shields.io/badge/Share-Facebook?style=for-the-badge&logo=facebook&colorB=555
[facebook-share-url]: https://www.facebook.com/sharer/sharer.php?u=https://github.com/frango9000/hal-form-client
[twitter-share-shield]: https://img.shields.io/badge/Share-Twitter?style=for-the-badge&logo=twitter&colorB=555
[twitter-share-url]: https://twitter.com/intent/tweet?url=https://github.com/frango9000/hal-form-client&text=Check%20this%20project%20out
[email-share-shield]: https://img.shields.io/badge/-email-black.svg?style=for-the-badge&colorB=555
[email-share-url]: mailto:info@example.com?&subject=&cc=&bcc=&body=Check%20this%20project%20out%20https://github.com/frango9000/hal-form-client
