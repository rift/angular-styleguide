# Angular Styleguide

*Opinionated Angular styleguide of industry-best practices for teams by [@deavon](//linkedin.com/in/deavon), with input from [John Papa](twitter.com/John_Papa), [Todd Motto](twitter.com/toddmotto), and the AngularJS team.*

A standardized approach for developing Angular applications in teams. This styleguide touches on concepts, syntax, and conventions.

## Table of Contents

1. [Modules](#modules)
1. [TypeScript & ES6](#typescript-and-es6)
1. [Lodash](#lodash)
1. [Controllers](#controllers)
1. [Components](#components)
1. [Services and Factories](#services-and-factories)
1. [Directives](#directives)
1. [Filters](#filters)
1. [Performance](#performance)
1. [Angular wrapper references](#angular-wrapper-references)
1. [Minification and annotation](#minification-and-annotation)
1. [Code Patterns](#code-patterns)
	1. [Single Responsibility](#single-responsibility)
	1. [Naming](#naming)
	1. [Folders-by-Feature Structure](#folders-by-feature-structure)
	1. [Application Structure LIFT Principle](#lift)
	1. [Application Structure](#application-structure)
	1. [Misc](#misc)
1. [Official TypeScript, Angular, & Lodash Docs](#official-typescript-angular-lodashdocs)

----------

## Modules

  - **Definitions**: Declare modules without a variable using the setter and getter syntax, and always use `export default` for `angular.module` to guarantee single responsibility and allow import into other module files.

    ```typescript
    /* avoid */
    var app = angular.module('app.subModule', []);
    app.controller();
    ```
    ```typescript
    /* recommended */
	export default angular.module('app.subModule', [])
		.controller();
    ```

    **Note**: Using `angular.module('app', []);` sets a module, whereas `angular.module('app');` gets the module. Only set once, and get for all other instances.

  - **Config files**: Once a module's config becomes as large or larger than its module file, always separate this into its own `*.config.ts` file. Also, **always** place the config and module files within the same directory.

	```typescript
	/* avoid */
	// settings.module.ts
	import {UserService} from './user.service';

	export default angular.module('app.settings', [])
		.service('userService', UserService)
		/* @ngInject */
		.config((
			$stateProvider: ng.ui.IStateProvider
		) => {
			// 30+ lines of code
		});
	```
	```typescript
	/* recommended */

	// settings.config.ts
	/* @ngInject */
	export default (
		$stateProvider: ng.ui.IStateProvider
	) => {
		// 30+ lines of code
	}
	
	// settings.module.ts
	import {UserService} from './user.service';
	import SettingsConfig from './settings.config';

	export default angular.module('app.settings', [])
		.service('userService', UserService)
		.config(SettingsConfig);
	```

  - **Methods**: Pass ES6 class references into module methods rather than assigning as a function callback

    ```typescript
    /* avoid */
    angular.module('app', [])
		.controller('MainController', function MainController () {

		})
		.service('SomeService', function SomeService () {

		});
	```
	```typescript
	/* recommended */
	// main.controller.ts
	export class MainController
	{
	}

	// some.service.ts
	export class SomeService
	{
	}

	// app.module.ts
	export default angular.module('app', [])
		.controller('MainController', MainController)
		.service('SomeService', SomeService);
    ```

    **Why?**: This aids with readability and reduces the volume of code "wrapped" inside the Angular framework

**[Back to top](#table-of-contents)**

----------

## TypeScript & ES6

  - **Third-party module definition import file**: Use a global definition file (i.e. `all.d.ts`) within your project's [`tsconfig.json`](https://github.com/Microsoft/TypeScript/wiki/tsconfig.json) to make definitions, for all plain-JavaScript third-party libraries, implicitly available to all code files in your project.

  Also, as unit tests are executed within their own separate context, always create a definition file only for your tests (i.e. `all.spec.d.ts`).

	```typescript
	/* avoid */
	/// <reference path="../typings/angularjs/angular.d.ts"/>

	export class DashboardController
	{
		/* @ngInject */
		constructor(private $q: ng.IQService)
		{
			// initialize controller
		}
	}
	```

	```javascript
	/* recommended */
	// tsconfig.json
	{
		"files": [
			"all.d.ts"
		]
	}
	```
	```typescript
	// all.d.ts
	/// <reference path="../typings/angularjs/angular.d.ts"/>
	```
	```typescript
	// dashboard.controller.ts
	export class DashboardController
	{
		/* @ngInject */
		constructor(private $q: ng.IQService)
		{
			// initialize controller
		}
	}
	```

  - **Angular dependency injector functions**: To limit redundancy, always specify the `private` accessor for all arguments of any Angular-based class' constructor. This will register these as properties within your class. It's best to keep injected dependency objects `private`.

    **Remember:** When resolving dependencies through Angular's DI mechanism, sort the dependencies by their type - the built-in AngularJS dependencies should be first, followed by your custom ones.

    **Remember:** For Angular DI arguments, always specify their corresponding type (i.e. `$q: ng.IQService` or `customService: CustomService`). For all other function declarations, specify argument types whenever possible.

    ```typescript
    /* avoid */
	import {UserService} from './user.service';

    export class DashboardController
    {
		private $q;
		private userService;

		/* @ngInject */
		constructor(userService, $)
	    {
			this.$q = $q;
			this.userService = userService;
	    }
    }
    ```
    ```typescript
    /* recommended */
	import {UserService} from './user.service';

    export class DashboardController
    {
		/* @ngInject */
	    constructor(
		    private $q: ng.IQService,
		    private userService: UserService
		) {
			// initialize controller
	    }
    }
    ```

  - **Type safety**: Prioritize the type safety of core components, like common Services, over one-time use Filters, Directives, etc. Try to share as many types as possible, and define or declare them within the scope of their use or in a place that makes sense for the whole app. Do not rush type safety.

    **Why?**: The more type safe properties, arguments, and method return types are specified, the less your application will be prone to errors, and the easier your code will be to manage. Type safety also improves code readability.

  **Remember**: If in doubt, use the `any` type. You can always add typing in later iterations. It's better to wait to code something the right way than to need to course correct somewhere down the road because something was rushed. Come to a consensus with your team on what typing structure and standard makes sense.

  - **Use implicit accessors**: For cleaner, more readable code, understand that TypeScript assumes a property or function is `public` when no accessor is specified. Only specify the accessor for `private` or `protected` class members.

    ```typescript
    // avoid
    export class DashboardController
    {
        private campaigns: Campaign[];
        public $q: ng.IQService;
		public filters: FilterTypes;
		public preloadHiddenTiles: boolean = true;

		/* @ngInject */
	    constructor($q: ng.IQService)
	    {
			this.$q = $q;
	    }

		public open()
		{
		}
    }
    ```
    ```typescript
    // recommended
    export class DashboardController
    {
        private campaigns: Campaign[];
		filters: FilterTypes;
		preloadHiddenTiles: boolean = true;

		/* @ngInject */
	    constructor(private $q: ng.IQService)
	    {
			// initialize controller
	    }

		open()
		{
		}
    }
    ```

    **Why?**: It improves code readability and reduces clutter.

    **Remember**: The less redundant syntax, the better.

  - **Hoisting**: In JavaScript, functions and variables are **hoisted**. Hoisting is JavaScript's behavior of moving declarations to the top of a [scope](https://developer.mozilla.org/en-US/docs/Glossary/scope) (the global scope or the current function scope).

  **example coming soon**

    **Note**: ES6 Classes are not hoisted, which will break your code if you rely on hoisting

    **Note:** An important distinction between function declarations and class declarations is that function declarations are hoisted and class declarations are not. You first need to declare your class and then access it, otherwise code like the following will throw a ReferenceError:

  **example coming soon**

  - **Use default exports sparingly**: Ensures that class and object names are identical throughout your source code, and limits confusion. `export default` is safe to use for the single-purpose files of anonymous functions, `*module.ts`, or Filters.

	```typescript
	/* avoid */
	// settings.controller.ts
	export default class SettingsController {}

	/* avoid */
	// maps-api.service.ts
	export default class MapsApiService {}

	// profile.module.ts
	import ArbitrarySettingsName from './settings.controller';
	import DifferentApiName from './maps-api.service';

	angular.module('app', [])
		.controller('SettingsController', ArbitrarySettingsName)
		.service('MapsApiService', DifferentApiName);
	```
    ```typescript
    /* recommended */
	// settings.controller.ts
	export class SettingsController {}

    /* recommended */
	// maps-api.service.ts
	export class MapsApiService {}

	// profile.module.ts
	import {SettingsController} from './settings.controller';
	import {MapsApiService} from './maps-api.service';

	angular.module('app', [])
		.controller('SettingsController', SettingsController)
		.service('MapsApiService', MapsApiService);

    ```

  - Avoid or limit the use of experimental TypeScript, or ES7 features, like @decorators.

**[Back to top](#table-of-contents)**

----------

## Lodash

  - **Third-party helper function priority**: There's some overlap in helper functions available between plain JavaScript, Angular, and Lodash. In the event that you need to use one of these functions, always opt for the [Lodash](https://lodash.com/docs) method first, then its [Angular](https://docs.angularjs.org/api/ng/function) equivalent second, and lastly, [plain JavaScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects):

	```typescript
	/* avoid */
	Object.keys(this.users).forEach(key => {
		var user = users[key];
		// Do something
	});
	```
	```typescript
	/* avoid */
	angular.forEach(this.users, (user, key) => {
		// Do something
	});
	```
    ```typescript
    /* recommended */
	_.each(this.users, user => {
		// Do something
	});
    ```

**[Back to top](#table-of-contents)**

----------

## Controllers

  - **controllerAs syntax**: Controllers are classes, so use the `controllerAs` syntax at all times

    ```html
    <!-- avoid -->
    <div ng-controller='MainController'>
		{{ someObject }}
    </div>
    ```
    ```html
    <!-- recommended -->
    <div ng-controller='MainController as vm'>
		{{ vm.someObject }}
    </div>
    ```

  - In the DOM we get a variable per controller, which aids nested controller methods, avoiding any `$parent` calls

  - The `controllerAs` syntax uses `this` inside controllers, which gets implicitly bound to `$scope`.

    ```typescript
    /* avoid */
    export class MainController
    {
		constructor(private $scope: ng.IScope)
		{
			this.$scope.someNumber = 0;
			this.$scope.doSomething = function () {
				this.$scope.someNumber = 5;
			};
		}
	}
    ```
    ```typescript
    /* recommended */
    export class MainController
    {
		someNumber = 0;

		doSomething()
		{
			this.someNumber = 5;
		}
	}
    ```

  - Only use `$scope` in `controllerAs` when necessary; for example, publishing and subscribing events using `$emit`, `$broadcast`, `$on`, or `$watch`. Try to limit the use of these, however, and treat `$scope` as a special use case:

    ```typescript
    /* recommended */
    export class MainController
    {
	    someNumber: boolean;

		doSomething()
		{
			this.someNumber = 5;
		}
	}
    ```

  - **Presentational logic only (MVVM)**: Presentational logic only inside a controller, avoid Business logic (delegate to Services)

    ```typescript
    /* avoid */
    export class MainController
    {
		users: User[];

		constructor(private $http: ng.IHttpService)
		{
			this.init();
		}

		init()
		{
			this.$http.get('/users').success((response: Users[]) => {
				this.users = response;
			});
		}
    }
    ```
    ```typescript
    /* recommended */
    import {UserService} from './user.service';

    export class MainController
    {
		users: User[];

		constructor(private userService: UserService)
		{
			this.init();
		}

		init()
		{
			this.userService.getUsers()
				.then((response: Users[]) => {
					this.users = response;
				});
		}
    }
    ```

    **Why?** : Controllers should fetch Model data from Services, avoiding any "business logic". Controllers should act as a ViewModel, and control the data flowing between the Model and the View presentational layer.

  *Placing business logic within your Controllers makes testing Services impossible.*

**[Back to top](#table-of-contents)**

----------

## Components

### Defer Logic to Services

  - Defer logic in a component by delegating to services.

    **Why?**: Logic may be reused by multiple components when placed within a service and exposed via a function.

    **Why?**: Logic in a service can more easily be isolated in a unit test, while the calling logic in the component can be easily mocked.

    **Why?**: Removes dependencies and hides implementation details from the component.

    **Why?**: Keeps the component slim, trim, and focused.

  **example coming soon**

### Keep Components Focused

  - Define a component for a view, and try not to reuse the component for other views. Instead, move reusable logic to factories and keep the component simple and focused on its view.

    **Why?**: Reusing components with several views is brittle, and good end-to-end (E2E) test coverage is required to ensure stability across large applications.

  **example coming soon**

**[Back to top](#table-of-contents)**

----------

## Services and Factories

  - All Angular Services are singletons, using `.service()` or `.factory()` differs in the way Objects are created.

  - **Services**: Act as a `constructor` function and are instantiated with the `new` keyword. Use `this` for public methods and variables

    ```typescript
    // some.service.ts
	export class SomeService
	{
		someSharedLogicMethod()
		{
		}
    }

	// app.module.ts
    angular.module('app', [])
		.service('SomeService', SomeService);
    ```

  - **Factories**: Use Factories sparingly.

  **AngularJS Team Note:** The use of `module.factory()` is specifically for when you are not using classes. The `module.service()` method was specifically designed for when you want to define your services as classes (or instantiable types). So there is actually no point in trying to hack together a way to register a class via the `module.factory()` method. Just use `module.service()` instead.

  **example coming soon**

    **Why?**: Primitive values cannot update alone using the revealing module pattern

**[Back to top](#table-of-contents)**

----------

## Directives

  - **DOM manipulation**: Takes place only inside Directives, never a Controller / Service. 
  
  **Never use `jQuery`**, or any third-party components dependent upon `jQuery`. Use JQLite instead with `angular.element` within Directives only.


    ```typescript
    /* avoid */
    // upload.controller.ts
    export class UploadController
    {
		constructor()
		{
			$('.dragzone').on('dragend', () => {
				// handle drop functionality
			});
		}
	}

	// common.module.ts
    import {UploadController} from './upload.controller';

    angular.module('app.common', [])
		.controller('UploadController', UploadController);
    ```
    ```typescript
	/* recommended */
	// drag-upload.directive.ts
	export function dragUploadDirective(): ng.IDirective {
		return {
			link: function (scope: ng.IScope, element: ng.IAugmentedJQuery) {
				element.on('dragend', () => {
					// handle drop functionality
				});
			}
		};
    }

	// common.module.ts
    import {dragUploadDirective} from './drag-upload.directive';

    angular.module('app')
      .directive('dragUpload', dragUploadDirective);
    ```

    **Why?**: Proper directive use enforces the separation of concerns and DRY principles very effectively, and most importantly, prevents the DOM from losing sync with your app's state. reduces the number of total forms of state management in your app.

  - **Declaration restrictions**: Only use `custom element` and `custom attribute` methods for declaring your Directives (`{ restrict: 'EA' }`) depending on the Directive's role

    **Why?**: Comment and class name declarations are confusing and should be avoided. Comments do not play nicely with older versions of IE. Using an attribute is also the safest method for browser coverage.

  - **controllerAs**: Use the `controllerAs` syntax inside Directives as well

    ```typescript
	/* avoid */
    // drag-upload-directive.controller.ts
	export class DragUploadDirectiveController
	{
		dragCompleted: boolean;
	}
    
    // drag-upload.directive.ts
    export function dragUpload () {
		return {
			controller: DragUploadController
		};
	}

    // common.module.ts
    angular.module('app.common', [])
		.directive('dragUpload', dragUpload);
    ```
    ```typescript
    /* recommended */
    // drag-upload-directive.controller.ts
	export class DragUploadDirectiveController
	{
		dragCompleted: boolean;
	}

    // drag-upload.directive.ts
	export function dragUpload () {
		return {
			controllerAs: 'vm',
			controller: DragUploadController
		};
	}

    // common.module.ts
	angular.module('app.common', [])
		.directive('dragUpload', dragUpload);
    ```

  - **Use the Controller syntax**: When your Directive's link function grows beyond a few lines of code.

	```typescript
    // color-swatch.directive.ts
	import {SomeService} from '../services/some.service';

	class ColorSwatchController
	{
		color: Color;

		/* @ngInject */
		constructor(
			private $element: ng.IAugmentedJQuery,
			private $attrs: ng.IAttributes,
			private someService: SomeService
		) {
			// Link function
		}

		onChange()
		{
			this.$element.css(...);
			this.someService.doWork();
		}
	}

	/* @ngInject */
	export function colorSwatchDirective(): ng.IDirective
	{
		return {
			bindToController: true,
			controller: ColorSwatchController,
			controllerAs: 'vm',
			replace: true,
			restrict: 'E',
			scope: {
				color: '=',
			},
			template: require('./color-swatch.html')
		}
	}
	```



**[Back to top](#table-of-contents)**

----------

## Filters

  - **Keep it light**: Make your filters as light as possible. They are called often throughout the `$digest` loop, so creating a slow filter will reduce app performance significantly.

  - **Global filters**: Create global filters using `angular.module('', []).filter()` syntax only. Never use local filters inside Controllers / Services

    ```typescript
    /* avoid */
    export class SomeController
    {
		startsWithLetterA(items: SomeItemType[])
		{
			return items.filter((item: SomeItemType) => {
				return /^a/i.test(item.name);
			});
		};
	}

    angular.module('app', [])
		.controller('SomeController', SomeController);
    ```
    ```typescript
    /* recommended */
    // starts-with-letter-a.filter.ts
	export function startsWithLetterAFilter()
    {
		return (items: SomeItemType[]) => {
			return items.filter((item: SomeItemType) => {
				return /^a/i.test(item.name);
			});
		};
	}

    // starts-with-letter-a.filter.ts
    import {startsWithLetterAFilter} from './starts-with-letter-a.filter.ts';

    angular.module('app.common', [])
		.filter('startsWithLetterA', startsWithLetterAFilter);
    ```

    **Why?:** This enhances testing and reusability

**[Back to top](#table-of-contents)**

----------

## Performance

  - **One-time binding syntax**: In versions of Angular (>= v1.3), use the one-time binding syntax `{{ ::value }}` everywhere you possibly can

    ```html
    <!-- avoid -->
    <h1>{{ vm.title }}</h1>
    <h1>{{ "global.default_title" | translate }}</h1>
    ```
    ```html
    <!-- recommended -->
    <h1>{{ ::vm.title }}</h1>
    <h1>{{ ::"global.default_title" | translate }}</h1>
    ```

  **Why?**: Binding once removes the watcher from the scope's `$$watchers` array after the `undefined` variable becomes resolved, thus improving performance of each dirty-check
    
  - **$scope.$digest**: Use `$scope.$digest` over `$scope.$apply`, where it makes sense.

    ```typescript
	$scope.$digest();
    ```

  **Why?**: `$scope.$apply` calls `$rootScope.$digest`, which causes the entire application `$$watchers` to dirty-check again. Using `$scope.$digest` will dirty check only the current and child scopes from the initiated `$scope`

**[Back to top](#table-of-contents)**

----------

## Angular wrapper references

  - **$document and $window**: Use `$document` and `$window` at all times to aid testing and Angular references

  - **$timeout and $interval**: Use `$timeout` and `$interval` over their native counterparts to keep Angular's two-way data binding up to date

    ```typescript
    /* avoid */
    export function dragUploadDirective() {
      return {
        link: function ($scope, $element, $attrs) {
          setTimeout(function () {
            //
          }, 1000);
        }
      };
    }
    ```
    ```typescript
    /* recommended */
    export function dragUploadDirective($timeout: ng.ITimeoutService) {
      return {
        link: ($scope: any, $element: ng.IAugmentedJQuery, $attrs: ng.IAttribute) => {
          $timeout(() => {
            //
          }, 1000);
        }
      };
    }
    ```

**[Back to top](#table-of-contents)**

----------

## Minification and annotation

  - **ng-annotate**: Use [ng-annotate](//github.com/olov/ng-annotate) for Gulp as `ng-min` is deprecated, and comment functions that need automated dependency injection using `/* @ngInject */`

    ```typescript
    export class MainController
    {
		/* @ngInject */
	    constructor(private $log: ng.ILogService)
	    {
			this.$log.debug('MainController loaded.');
	    }
    }
    ```

  - Which produces the following output with the `$inject` annotation

    ```typescript
	function e(e){this.$log=e,this.$log.debug("MainController loaded.")}return e.$inject=["$log"],e
    ```


**[Back to top](#table-of-contents)**

----------

## Code Patterns

### Single Responsibility

#### Rule of 1

  - **Define 1 component per file.**

  **Why?**: One component per file promotes easier unit testing and mocking.

  **Why?**: One component per file makes it easier to read, maintain, and avoid collisions with teams in source control.

  **Why?**: One component per file avoids hidden bugs that often arise when combining components in a file where they may share variables, create unwanted closures, or unwanted coupling with dependencies.

  The following example defines the `app.profile` module and its dependencies, defines a controller, and defines a service all in the same file.

	```typescript
	/* avoid */
	// profile.ts
	class UserProfileController
	{
	}

	class UserService
	{
	}

	angular.module('app.profile', [])
		.controller('UserProfileController', UserProfileController)
		.service('userService', UserService);
	```

  The same components are now separated into their own files.

	```typescript
	/* recommended */
	// user-profile.controller.ts
	export class UserProfileController
	{
	}

	// user.service.ts
	export class UserService
	{
	}

	// profile.module.ts
	import {UserProfileController} from './user-profile.controller';
	import {UserService} from './user.service';

	angular.module('app.profile', [])
		.controller('UserProfileController', UserProfileController)
		.service('userService', UserService);
	```

**[Back to top](#table-of-contents)**

#### Small Functions

  - Define small functions, no more than 75 LOC (less is better).

  **Why?**: Small functions are easier to test, especially when they serve one, distinct purpose.

  **Why?**: Small functions promote reuse.

  **Why?**: Small functions are easier to read.

  **Why?**: Small functions are easier to maintain.

  **Why?**: Small functions help avoid hidden bugs that come with large functions that share variables with external scope, create unwanted closures, or unwanted coupling with dependencies.

**[Back to top](#table-of-contents)**

#### IIFE

  - Given full adherence to the Single Responsibility principle, modern build tooling (i.e. webpack), and ECMAScript 6 (ES6) modules, manually wrapping your code in immediately-invoked function expressions (IIFE) or closures is no longer necessary. Webpack, Browserify, and other tools already do this for you at build time, and is the superior mechanism moving forward.

	```typescript
	/* avoid */
	(function() {
		'use strict';

		class PersistentCacheService {}

		angular.module('app.common', [])
			.service('persistentCache', PersistentCacheService);
	})();
	```
	```typescript
	/* avoid */
	export default function(ngModule) {

		/*@ngInject*/
		function AnalyticsController($scope, $state) {
		}

		ngModule.controller('analyticsController', AnalyticsController);
	}
	```

  **Why?**: The less syntax redundancy you can have in your codebase, the less time must be spent on maintenance and the easier your code will be to read and understand.

**[Back to top](#table-of-contents)**

----------

### Naming

#### File Naming Conventions

  - Use consistent names for all components following a pattern that describes the component's feature then (optionally) its type. The ideal pattern is `feature-name.type.ts`.

  **Why?**: Naming conventions help provide a consistent way to find content at a glance. Consistency within the project is vital. Consistency with a team is important. Consistency across a company provides tremendous efficiency.

  **Why?**: Provides a consistent way to quickly identify components.

  **Why?**: Provides pattern matching for any automated tasks.

There are 3 naming conventions for most assets:

Asset Type |  File Name  |  Angular Token  |  Class or Function Name  |
--------- | -------------------------- | --------------------- | --------------------- |
Controller | feature-name.controller.ts | featureNameController | FeatureNameController |
Service    | feature-name.service.ts    | featureNameService    | FeatureNameService |
Component  | feature-name.component.ts  | featureNameComponent  | FeatureNameComponent |
Directive  | feature-name.directive.ts  | featureName           | featureNameDirective() |
Factory    | feature-name.factory.ts    | featureNameFactory    | featureNameFactory() |
Filter     | feature-name.filter.ts     | featureName           | featureNameFilter() |

**[Back to top](#table-of-contents)**

----------

### Folders-by-Feature Structure

  - Create folders named for the feature they represent. When a folder grows to contain more than 7 files, start to consider creating a folder for them. Your threshold may be different, so adjust as needed.

    **Why?**: A developer can locate the code, identify what each file represents at a glance, the structure is flat as can be, and there are no repetitive nor redundant names.

    **Why?**: The LIFT guidelines are all covered.

    **Why?**: Helps reduce the app from becoming cluttered through organizing the content and keeping them aligned with the LIFT guidelines.

    **Why?**: When there are a lot of files (10+) locating them is easier with a consistent folder structures and more difficult in flat structures.

  **example coming soon**

**[Back to top](#table-of-contents)**

----------

### Application Structure LIFT Principle

#### LIFT

  - Structure your app such that you can `L`ocate your code quickly, `I`dentify the code at a glance, keep the `F`lattest structure you can, and `T`ry to stay DRY. The structure should follow these 4 basic guidelines.

    **Why LIFT?**: Provides a consistent structure that scales well, is modular, and makes it easier to increase developer efficiency by finding code quickly. Another way to check your app structure is to ask yourself: How quickly can you open and work in all of the related files for a feature?

    When I find my structure is not feeling comfortable, I go back and revisit these LIFT guidelines

    1. `L`ocating our code is easy
    2. `I`dentify code at a glance
    3. `F`lat structure as long as we can
    4. `T`ry to stay DRY (Don’t Repeat Yourself) or T-DRY

#### Locate

  - Make locating your code intuitive, simple, and fast.

    **Why?**: I find this to be super important for a project. If the team cannot find the files they need to work on quickly, they will not be able to work as efficiently as possible, and the structure needs to change. You may not know the file name or where its related files are, so putting them in the most intuitive locations and near each other saves a ton of time. A descriptive folder structure can help with this.

  **example coming soon**

#### Identify

  - When you look at a file you should immediately know what it contains and represents.

    **Why?**: You spend less time hunting and pecking for code, and become more efficient. Even if it means having longer file names. Be descriptive with file names, and keep the contents of the file to exactly 1 component. Avoid files with multiple controllers, multiple services, or a mixture. There are rare exceptions to the 1-per-file rule in instances of very small but closely-related features, after which the file is still easily identifiable.

#### Flat

  - Keep a flat folder structure as long as possible. When you get to 7+ files, begin considering separation.

    **Why?**: Nobody wants to search 7 levels of folders to find a file. Think about menus on web sites … anything deeper than 2 should take serious consideration. In a folder structure there is no hard and fast number rule, but when a folder has 7-10 files, that may be time to create subfolders. Base it on your comfort level. Use a flatter structure until there is an obvious value (to help the rest of LIFT) in creating a new folder.

#### T-DRY (Try to Stick to DRY)

  - Be DRY, but don't go nuts and sacrifice readability.

    **Why?**: Being DRY is important, but not crucial if it sacrifices the others in LIFT.

    **example coming soon**

**[Back to top](#table-of-contents)**

----------

### Application Structure

#### Modularity

##### Many Small, Self Contained Modules

  - Create small modules that encapsulate one responsibility.

    **Why?**: Modular applications make it easy to plug and go as they allow the development teams to build vertical slices of the applications and roll out incrementally. This means we can plug in new features as we develop them.

    **example coming soon**

**[Back to top](#table-of-contents)**

----------

### Others

  - The less unique forms of state management you can have throughout your app, the better. Some state management examples include: CSS visibility and DOM manipulation outside of Directives (very, very bad), local variables, Controller variables, DOM manipulation through directives, any server-side output changes, DOM manipulation outside of directives, etc. You can easily see how one form could contradict or conflict with another form, and ultimately lead to your app code losing sync with the DOM and causing some fatal error.
  - Use `$resource` instead of `$http` when possible. The higher level of abstraction will save you from redundancy.
  - Never use globals. Resolve all dependencies using Dependency Injection; this will prevent bugs and monkey patching when testing.
  - Think twice when working with `$rootScope`, or potentially polluting it.

**[Back to top](#table-of-contents)**

----------

## Official TypeScript, Angular, & Lodash Docs

For anything else, including API reference, check the official documentation:

- [TypeScript](//www.typescriptlang.org/Handbook)
- [Angular](//docs.angularjs.org/api)
- [Lodash](//www.lodash.com/docs)

----------

## Contributing

	1. Discuss the change in a GitHub issue.
	2. Open a Pull Request, reference the issue, and explain the change and why it adds value.
	3. The Pull Request will be evaluated and either merged or declined.
