# Car Voting System - Angular Routing Workshop

## Learning Objectives

- Build a more complex angular app with multiple components
- Use Angular Routing in the application
- Pass data between components
- Use parameters in the url to specifiy which component to display

## Getting started

Create a new Angular project using:

```bash
ng new angular-car-voting --no-standalone
```

just hit enter to accept the defaults.

Switch into the new project and open it with VS Code. Then open a terminal and run the application:

```bash
ng serve
```

## Adding various components

Change the title in `index.html` to be something more human-readable. Delete `app.component.spec.ts` as we aren't going to use it. Go into `app.component.ts` and change the value of title to be something nicer (you can also specify that it will be a string too). Delete the contents of `app.component.html` and replace it with

```html
<h1>{{title}}</h1>

<router-outlet />
```

We are going to use `router-outlet` to do our routing later on.

Now we're going to add some components, services, modules and a layout. The modules appear to work as a way of grouping related components together, but Angular going forward may also be moving away from this paradigm.

Add a cars service, this will work to load the data and provide various methods that we will use later in our application. This works in a similar way to the Repository code we generated in Spring.

```bash
ng generate service cars --skip-tests
```

Then generate a cars module to hold the various components the cars part is going to have.

```bash
ng generate module cars
```

Followed by calls to generate `add`, `view`, `list` and `edit` components inside the cars module:

```bash
ng generate component cars/add --skip-tests --no-standalone
```

```bash
ng generate component cars/view --skip-tests --no-standalone
```

```bash
ng generate component cars/list --skip-tests --no-standalone
```

```bash
ng generate component cars/edit --skip-tests --no-standalone
```

Then we'll make a layout module to hold a menu component

```bash
ng generate module layout
```

and

```bash
ng generate component layout/menu --skip-tests --no-standalone
```

Have a look through all of the components and modules that have just been generated.

## Adding some routes and lots of imports/exports

Find the `app-routing.module.ts` file and open it. You need to add the routes we are going to use in the application to it inside the routes array:

```typescript
const routes: Routes = [
  {
    path: "cars",
    component: ListComponent,
  },
  {
    path: "cars/add",
    component: AddComponent,
  },
  {
    path: "cars/:id",
    component: ViewComponent,
  },
];
```

You will need to add imports for each of those components. Notice the last one is going to take in an `id` value which we'll use to select the car object we want to display. We'll discuss how to do this later.

Next you need to find the `layout.module.ts` file and open it. In the file you need to add `RouterModule` to the imports section and an exports section that includes MenuComponent. Make sure that suitable import statements get generated at the top of the file.

```typescript
@NgModule({
  declarations: [MenuComponent],
  imports: [CommonModule, RouterModule],
  exports: [MenuComponent],
})
export class LayoutModule {}
```

Next open the `menu.component.html` file and add the following:

```html
<h2>Menu</h2>
<ul>
  <li><a routerLink="/">Home</a></li>
  <li><a routerLink="/cars">Cars</a></li>
  <li><a routerLink="/cars/add">Add a Car</a></li>
</ul>
```

Then open `app.module.ts` and add the modules we've created along with the `CommonModule` to the imports section:

```typescript
  imports: [
    BrowserModule,
    AppRoutingModule,
    CommonModule,
    LayoutModule,
    CarsModule
  ],
```

again you'll need to make sure they're correctly imported. Next open the `cars.modules.ts` file and makes sure that the all of the components we created in the cars module are correctly declared, imported and exported. Similar to what we did in the layout modules file.

```typescript
@NgModule({
  declarations: [
    AddComponent,
    ViewComponent,
    ListComponent,
    EditComponent],
  imports: [
    CommonModule,
    RouterModule],
  exports: [
    AddComponent,
    ViewComponent,
    ListComponent,
    EditComponent],
})
```

The RouterModule will need to be imported at the top of the file too, but otherwise the imports are probably already sorted.

Go into `app.component.html` and add in the menu component between the heading and the router-outlet

```html
<h1>{{title}}</h1>

<app-menu></app-menu>
<router-outlet />
```

## Creating a Car Model and some dummy data

Click on the cars folder on the left and add a new folder called `models` then inside that folder add a file called `car.ts` and add the following interface to it:

```typescript
export interface Car {
  id: number;
  make: string;
  model: string;
  description: string;
}
```

This specifies the shape that all of our car objects will take when we start to generate them. Next rename the original `cars.service.ts` to be `car.service.ts` and also rename any reference to it inside the file. Next add a line at the top to import the Car model we created.

```typescript
import { Car } from "./cars/models/car";
```

Then go into the cars folder and add another folder, this time called `data` and then inside there, add a file called `cars.ts` which is where we'll put some initial dummy data:

```typescript
import { Car } from "../models/car";

export const CARS: Car[] = [
  {
    id: 1,
    make: "Porsche",
    model: "911",
    description: "Archetypal sportscar",
  },
  {
    id: 2,
    make: "Volks Wagen",
    model: "Beetle",
    description: "Herbie",
  },
];
```

If we were actually getting our data from an API then these last steps would probably not be needed as we could just pull it in from there. Although our model might still need to exist but would need to match the API data we get back.

Next let's go back in to `car.service.ts` and add in that dummy data along with some functions that will act as endpoints when we access them via various paths in the application. Delete the constructor and add in the following two lines:

```typescript
  private currentId: number = 1;
  private cars: Car[] = CARS;
```

The first one we're going to use to control the id values of our Car objects and the second one loads the dummy data into an array of Cars objects called `cars`.

Next add the following two methods to return either a single car, which we find by id or all of the cars. If we pass in an invalid id then we will return null, but we might also potentially pass a null id in, so the signature of the first method covers all of those eventualities.

```typescript
  public getCarById(id: number | null): Car | null {
    const car = this.cars.find((car) => car.id === id);
    if (!car) {
      return null;
    }
    return car;
  }

  public getAllCars(): Car[] {
    return this.cars;
  }
```

## Adding a list of all of the cars

Let's make it so that when we go to the `cars` endpoint the List Component gets returned and shows us a list of all of the cars we currently have stored. To do that we need to get our CarService into the ListComponent class and then use it to get all of the cars into a variable. Add the following into the class definition in `list.component.ts`, to inject the CarService in and then load the list of cars from it.

```typescript
export class ListComponent {
  carService = inject(CarService);

  cars: Car[] = this.carService.getAllCars();
}
```

Make sure you get the imports to work correctly too.

Open up the `list.component.html` file and add the following code in there:

```html
<h2>Cars List Page</h2>
<table>
  <thead>
    <tr>
      <th>Id</th>
      <th>Make</th>
      <th>Model</th>
      <th>Description</th>
      <th>Link</th>
    </tr>
  </thead>
  <tbody>
    <tr *ngFor="let car of cars">
      <td>{{car.id}}</td>
      <td>{{car.make}}</td>
      <td>{{car.model}}</td>
      <td>{{car.description}}</td>
      <td><a routerLink="/cars/{{ car.id }}">Link</a></td>
    </tr>
  </tbody>
</table>
```

Check whether you can see the cars table at this point, if not then you'll need to debug what is happening in your code to fix this.

If you look in `app.component.html` you should see tags that look like: `<router-outlet></router-outlet>` or possibly `<router-outlet />` and this is what the app routing module resolves into the component which matches the url as defined in the routes variable.

## Add code to allow viewing a car using its ID value

Find the `view.component.ts` file and add code to inject a CarService object and an ActivatedRoute object into the class. The ActivatedRoute gives us access to the URL that was used to navigate to this current page (ie the url including :id at the end):

```typescript
carService = inject(CarService);
route = inject(ActivatedRoute);
```

then just below these we want to use the route to extract the id into a variable and then pass that into the carService object's getCarById method.

```typescript
  id = this.route.snapshot.paramMap.get('id')
  car: Car | null = this.carService.getCarById(Number(this.id))
```

Make sure you allow the imports to autocomplete to pull in the correct code.

Then open the `view.component.html` file to add in code to use these values.

```html
<ng-container *ngIf="car === null; else elseBlock">
  <div>
    <p><strong>Car not available</strong></p>
  </div>
</ng-container>
<ng-template #elseBlock>
  <div>
    <h1>{{car?.make}} {{car?.model}}</h1>
    <p>{{car?.description}}</p>
  </div>
</ng-template>
```

The `?` after each use of car in the bottom part is to indicate that car could be null (even though the if protects against that situation). An alternative solution is to remove the `else elseBlock` indicator and replace the `ng-template` part with another `ng-container` if block this time for `car !== null`.

## Use Forms to Add a new Car

Open `add.component.html` and add the following code to implement the form (there will be lots of red and yellow squiggles for now, we'll fix them shortly)

```html
<div>
  <h2>Add a car</h2>
  <form [formGroup]="carForm" (ngSubmit)="addCar()">
    <div>
      <label for="make">Make</label>
      <input type="text" id="make" formControlName="make" />
    </div>
    <div>
      <label for="model">Model</label>
      <input type="text" id="model" formControlName="model" />
    </div>
    <div>
      <label for="description">Description</label>
      <input type="text" id="descritpion" formControlName="description" />
    </div>
    <button type="submit">Add Car</button>
  </form>
</div>
```

This is just one of Angular's takes on how to build a form. Then in `add.component.ts` add the following code:

```typescript
export class AddComponent {
  carForm: FormGroup;
  formBuilder = inject(FormBuilder);

  constructor() {
    this.carForm = this.formBuilder.group({
      make: ["", Validators.required],
      model: ["", Validators.required],
      description: ["", Validators.required],
    });
  }
}
```

bringing in the correct imports along with them. The empty strings are the initial value for each text box and the Validators are set to make those fields required (there are lots of others and you will also be able to configure your own, somewhere). Once we have the rest of the code working you can go back and work out how to make use of these in order to prevent a car being submitted if the fields aren't valid.

We need to add a missing dependency to `cars.module.ts` change the imports section to be:

```typescript
  imports: [
    CommonModule,
    RouterModule,
    ReactiveFormsModule
  ],
```

Now back in the `add.component.ts` file we need to add an `addCar` method, which we'll use to gather the values from the form and pass them to the CarService where we'll need to add another method that will allow us to submit a new car to the cars list. Change the code so that it looks like this:

```typescript
export class AddComponent {
  carForm: FormGroup;
  formBuilder = inject(FormBuilder)
  carService = inject(CarService)
  router = inject(Router)

  myCar: Car = {id: 0, make: '', model: '', description: ''}

  constructor() {
    this.carForm = this.formBuilder.group({
      make: ['', Validators.required],
      model: ['', Validators.required],
      description: ['', Validators.required],
    });
  }

  addCar() {
    this.myCar.make = this.carForm.value.make,
    this.myCar.model = this.carForm.value.model,
    this.myCar.description = this.carForm.value.description
    
    this.carService.addCar(this.myCar)
    this.router.navigate(['cars'])
  }
}
```

The final line forces the browser to navigate straight back to the list page once the car has been added.

Then we need to do some work on the CarService class to add some helper functions. So open the `car.service.ts` file and change it slightly so that it looks like this

```typescript
export class CarService {
  private cars: Car[] = CARS;
  private currentId: number = this.cars.length;

  public getCarById(id: number | null): Car | null {
    const car = this.cars.find((car) => car.id === id);
    if (!car) {
      return null;
    }
    return car;
  }

  public getAllCars(): Car[] {
    return this.cars;
  }

  public addCar(car: Car) {
    this.currentId++; // Get a new value each time we add a car
    this.cars.push({ ...car, id: this.currentId }); // Set that value in the object we push
  }
}
```

## Exercises

Once you have this all working then here are a couple of things to try:

1. Can you add in the functionality to allow you to edit an existing car?
2. Can you add in validation so that you cannot submit your form whilst one of the text fields is blank?
3. Can you add in your own custom validations that you can use to prevent submissions unless certain validations are met?

