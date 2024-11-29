# wenroll_docs

### 1. Introduction
This document provides an overview of a mobile application built with React Native for the front-end, Node.js and NestJS for the back-end, and GraphQL as the API layer. The application is designed for managing user accounts, real-time data interactions, and seamless integration between client and server.
### 2. Technologies Used
The application uses the following technologies:
- **React Native**: For building cross-platform mobile applications.
- **Node.js**: As the runtime environment for the back-end.
- **NestJS**: A framework for building scalable server-side applications.
- **GraphQL**: For flexible and efficient data queries and mutations.
- **PostgreSQL**: For data storage.

### 3. Application Architecture
The application follows a three-tier architecture:
- **Presentation Layer**: React Native handles the user interface and client-side logic.
- **Business Logic Layer**: NestJS provides structured back-end development with support for GraphQL.
- **Data Layer**: PostgreSQL serves as the primary database.
Data flows between these layers using GraphQL APIs exposed by the NestJS back-end.
### 4. Setting Up the Application

### Prerequisites
1. Node.js (version 16.XX.X):

 - **Ensure that Node.js version 16.x or higher is installed. You can check your current version with the following command:**

``` 
node -v

```

- **If you need to install or update Node.js, download it from Node.js Official Website or use a version manager like nvm.**



2.  PostgreSQL Database:

- **Make sure PostgreSQL is installed and running. You can check the version with:**

```
psql --version
```

- **If not installed, download and install PostgreSQL from PostgreSQL Official Website.**

- **Ensure the database is properly configured and running. You can start PostgreSQL using:**
```
sudo service postgresql start
```
 ENV file
 ```
JWT_SECRET 
DATABASE
SENDGRID_KEY 
HOST 
S3_ACCESS_KEY 
S3_SECRET_KEY 
S3_MEDIA_CONVERT_ENV 
GOOGLE_KEY
S3_URL
BUCKET 
DB_NAME 
CLUSTER_NAME
PAYMENT_KEY 
VIDEO_CONVERTER_URL 
FIREBASE_TYPE
FIREBASE_PROJECT_ID
FIREBASE_KEY_ID
FIREBASE_PRIVATE_KEY
FIREBASE_CLIENT_EMAIL
FIREBASE_AUTH_URI
FIREBASE_TOKEN_URI
FIREBASE_AUTH_PROVIDER
FIREBASE_CLIENT_URL
FIREBASE_UNIVERSAL_DOMAIN
FIREBASE_CLIENT_ID
FIREBASE_DB
APPLE_PURCHASE_KEY
APPLE_ISSUER_ID
SANDBOX_URL
APPLE_URL
APPLE_PRIVATE_KEY
APPLE_CLIENT_ID
APP_DB_USERNAME
APP_DB_PASSWORD
APP_DB_PORT
APP_DB_DATABASE
APP_DB_DIALECT
APP_DB_HOST
APP_DB_SCHEMA
APP_DB_SSL
```

### Back-End Setup



1. Install dependencies:
   ```bash
   yarn install
   ```
2. Start the NestJS server:
   ```bash
   npm run start:dev
   ```
Ensure the PostgreSQL database is running and the connection parameters are correctly configured in the environment variables.
### 5. API Documentation

 ### *Auth Module*
- **Mutation createUser(input: SignUpInput): User**
Description: This endpoint allows the creation of a new user by providing necessary information like email, password, first name, and lastname. It registers the user and sends an invitation email with a temporary password. 

#### Request Body 
```
 mutation createUser($input: SignUpInput!){
	 createUser(input: $input){
		...UserFragment 
		 } 
	}
 ```

 #### Features
- **Email Validation: Ensures the provided email is not already registered with an active user. If a user with the same email exists, the registration fails with a conflict error..**
- **Password Management: If no password is provided, a temporary password is generated. The password is securely hashed before being stored.**
- **User Creation: A new user is created with the provided details, including role (STUDENT), status (PENDING), and the requirement to reset the password.**
– **Invitation Email: Sends an invitation email with a deep link that allows the user to reset their password and activate their account.**
- **Secure Token Generation: A reset token is generated and used in the invitation email to allow the user to reset their password securely**
 

| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------|
| **200** | `Success` | User created succesfully. |  
| **400** | `Bad Request` | The request was malformed or missing required parameters. | 
| **409** | `Conflict` | User with this email already registered| 



-  **Mutation auth(input: LoginInput): AuthPayload**

Description This route enables users to log in by providing their email and password. Upon successful authentication, the system generates an access token and returns it along with user details. The access token can be used for subsequent authenticated requests.
 #### Features
- **Validates the user's email and password credentials.**
- **Supports account suspension checks and handles login restrictions for non-active users.**
- **Updates the user's last login date upon successful login.**
- **Ensures secure password handling and token generation.**
#### Request Body 
```
 auth($input: LoginInput!) {
        auth(input: $input) { 
	accessToken 
	...UserFragment 
	 } 
}
 ```

| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------| 
| **200** | `Success` | The request was succesfuul, user is loged in the system. | 
| **400** | `Bad Request` | The request was malformed or missing required parameters. | 
| **401** | ` Unauthorized` |Authentication failed; invalid credentials provided or user is suspended | 


- **Mutation  appleAuth(input: AppleAuth): AuthPayload**

Description: This endpoint allows a user to authenticate using Apple’s identity token. Upon successful authentication, it checks the validity of the token, verifies the user’s email and other credentials, and either creates a new user or processes an already registered user. It also checks that the email associated with the token matches the email provided by the user.

#### Features 
- **Token Decoding and Verification: The endpoint decodes the identity token and verifies it using Apple's public key to ensure the authenticity of the token.**
- **Email Validation: Checks that the email from the decoded token matches the provided email. If not, an UnauthorizedException is thrown.**
- **User Creation: If the email is valid and not previously registered, the system creates a new user by calling the findOrCreate function with necessary details.**
- **Existing User Handling: If the user is already registered, the system looks up the user by sub (a unique identifier for the user in Apple’s system) and processes the authentication flow for registered users.**
- **Error Handling: Detailed error messages are returned for issues such as token verification failure, mismatched emails, or if the user is not found in the system.**

#### Request Body 

```
appleAuth($input: AppleAuth!) {
 appleAuth(input: $input) {
 	accessToken 
	…UserFragment
 }
 ```
| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------| 
| **200** | `Success` | Request was successful, user is regsitered and redirected into system.| 
| **401** | `Unauthorized` | User email and token are not for the same account| 
| **401** | ` Unauthorized` |The identity token is invalid or could not be verified. | 
| **404** | ` Not found` |No user was found with the provided identifier (sub from the decoded token). | 


 - **Mutation googleAuth(input: GoogleAuthInput): AuthPayload**
Description: This endpoint handles user authentication via Google. It validates the provided Google ID token, ensures it matches the user ID, and creates or retrieves the corresponding user account

#### Features 
- **Token Verification: The endpoint uses Google's verifyIdToken to authenticate the user’s token and ensure that it is valid**
-  **Google User ID Matching: It compares the Google user ID (sub from the token) with the user ID provided in the credentials to confirm that they match.**
- **User Creation: If the user is valid, a new user is created or found using the findOrCreate method.**
- **Error Handling: Proper error handling is implemented for cases such as invalid tokens, user ID mismatches, or errors during user creation.**


#### Request Body 

```
googleAuth($input: GoogleAuthInput!) {
	 googleAuth(input: $input) { 
		accessToken
		…UserFragment	
	 }
 }
```

| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------|
| **200** | `Success` | Request was successful, user is regsitered and redirected into system.| 
| **400** | `error verifying google idToken email:<user.email>` |The Google ID token could not be verified.| 
| **401** | ` Unauthorized` |The token and user ID do not match or do not correspond to the same user. | 
| **500** | `error creating google user with email:<user.email>` |A failure occurred while creating a new Google user account. | 



 ### *Users Module*

 - **Query userWithNotes(): User**

Description: This endpoint fetches users based on their role within the system. It allows filtering, pagination, and retrieval of all users if required. It also checks the role of the current user to enforce permission-based access.

#### Features
 - **User Retrieval: The endpoint fetches a user by their ID using the findById method.**
- **Notes Handling: It retrieves the user's notes and enriches each note with additional**
- **lesson information. Lesson Info Enrichment: For each note, the endpoint fetches lesson**
- **details such as the video link and associates it with the note. Error Handling: If the user**
- **does not exist or there is an issue with fetching the lesson data, appropriate errors are thrown.**

#### Request Body

``` 
userWithNotes { 
	…userFragment
 }
```
| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------| 
| **202** | `success` | The request was successful, and the users are returned. |
| **400**          | `Bad Request`         | The request parameters were invalid (e.g., missing required fields or invalid role). |
| **403**          | `Forbidden`           | The current user does not have permission to view the requested role.           |
| **404**          | `Not Found`           | The requested resource (e.g., user, data) was not found.                        |
| **401**          | `Unauthorized`        | The user is not authorized to access this resource or perform the operation.    |



 - **Query usersByRole(role: String!, filter: UserFilter, fetchAll: Boolean, currentPage: Int, perPage: Int): FetchUsersResponse**

Description: This endpoint fetches users based on their role within the system. It allows filtering, pagination, and retrieval of all users if required. It also checks the role of the current user to enforce permission-based access.

#### Features

- **Role-Based Filtering: The endpoint retrieves users that match the specified role (e.g., Admin, Moderator, etc.).**
- **Pagination: It supports pagination, allowing users to specify the current page and the number of users per page. If fetchAll is set to true, it retrieves up to 1000 users.**
- **User Permissions Check: The endpoint checks the current user's role, and certain actions are restricted based on the user's permissions (e.g., SuperAdmin cannot view Moderators).**
number of users per page. If fetchAll is set to true, it retrieves up to 1000 users.**
- **Company-Specific Filtering: Users can be filtered based on the companyId, and only active users are retrieved.**

#### Request Body

```
  usersByRole($role: String!, $filter: UserFilter, $fetchAll: Boolean, $currentPage: Int, $perPage: Int) {
	 usersByRole(role: $role, filter: $filter, fetchAll: $fetchAll, currentPage: $currentPage, perPage: $perPage) {
	 currentPage
	 totalPages 
	totalCount
	 data { 
	…userFragment 
		}
	 }
 }
```

| **Status Code** | **Message** | **Description** | 
|------------------|---------------------|----------------------------------------------------------------| 
| **202** | `success` |The request was successful, and the users are returned.| 
| **400** | `Bad Request` |The request parameters were invalid (e.g., missing required fields or invalid role).| 
| **403** | `Forbidden` |The current user does not have permission to view the requested role.| 

- **Query usersByRole(role: String!, filter: UserFilter, fetchAll: Boolean, currentPage: Int, perPage: Int): FetchUsersResponse**

Description: This endpoint allows the retrieval of users based on their role within the system. It supports filtering, pagination, and the option to fetch all users if needed. Additionally, it checks the role of the current user to ensure that the request is permitted based on their permissions.

#### Features

- **Role-Based Filtering: Retrieve users by specific roles (e.g., Admin, Moderator).**
- **The role parameter filters users based on their assigned role.**
- **Pagination: Supports pagination with the currentPage and perPage parameters, allowing**
- **control over the number of users retrieved.**
- **If fetchAll is set to true, up to 1000 users are returned.**
- **User Permissions Check: Verifies the current user's role and restricts actions if necessary.**
- **Filtering: Users can be filtered by companyId, and only active users are returned.**

#### Request Body

```
query usersByRole($role: String!, $filter: UserFilter, $fetchAll: Boolean, $currentPage: Int, $perPage: Int) { 
 	usersByRole(role: $role, filter: $filter, fetchAll: $fetchAll, currentPage: $currentPage, perPage: $perPage) { 
	currentPage 
	totalPages 
	totalCount 
	data {
	 ...UserFragment
		 }
	 }
 }
```

| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **202** | `success` | The request was successful, and the users are returned. |
| **400**          | `Bad Request`         | The request parameters were invalid (e.g., missing required fields or invalid role). |
| **403**          | `Forbidden`           | The current user does not have permission to view the requested role.           |
| **404**          | `Not Found`           | The requested resource (e.g., user, data) was not found.  |                  
| **401**          | `Unauthorized`        | The user is not authorized to access this resource or perform the operation.    |

- **Mutation updateUser(id: String, input: UserInput, avatar: Upload): User**

Description: This endpoint allows updating a user's profile. It accepts a user ID, new user details (like name, email, etc.), and an optional avatar upload. The user's profile is updated, and any invalid data (like an invalid mobile number) is handled gracefully.

### Features
- **User Profile Update: The endpoint updates the user's details such as name, email, mobile number, etc.**
- **Avatar Upload: Allows the user to upload a new avatar image, which is processed and updated in the user's profile.**
- **Validation: Ensures that input data, such as the mobile number, is valid.**
- **If the mobile number is invalid or below a threshold, it is removed.**
- **Role-Based Guarding: Only authorized users can access this endpoint, with guards to ensure proper access control.** 
- **Error Handling: Ensures that errors during the update process (such as invalid inputs or file uploads) are handled properly.**


#### Request Body

```
updateUser($id: String!, $input: UserInput!, $avatar: Upload) { 
 updateUser(id: $id, input: $input, avatar: $avatar) {
…userFragment 
	}
 }
```


| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **200** | `Success` | The request was successful, and the user profile was updated. | | **400** | `Bad Request` | The request parameters were invalid (e.g., missing required fields or invalid data). |
| **401** | `Unauthorized` | The user does not have permission to update this profile. | 
| **404** | `User Not Found` | The user with the provided ID was not found. |

- **Mutation addOrDeleteCategoryForStudent(ids: [String]): String**

Description: This endpoint allows managing categories for a student. Categories that already exist for the student are removed, while new categories from the input are added. It ensures efficient updates by checking existing associations and only making the necessary changes.


#### Features

- **Dynamic Category Management: Automatically adds new categories and removes existing ones for the student.**
- **Parallel Processing: Uses asynchronous operations to handle multiple categories efficiently.**
- **Role-Based Access Control: Protected by GqlAuthGuard to ensure only authorized users can make updates.**
- **Error Handling: Ensures that invalid inputs or internal errors are properly addressed and logged.**
#### Reqsuest Body
```
addOrDeleteCategoryForStudent($ids: [String]!) {
 addOrDeleteCategoryForStudent(ids: $ids)
 }
```

| **Status Code** | **Message**           | **Description**                                                     |
|------------------|-----------------------------|---------------------------------------------------------------------|
| **200**          | `Success`                  | The request was successful, and the categories were updated.       |
| **400**          | `Bad Request`              | Invalid or missing request parameters.                             |
| **401**          | `Unauthorized`             | The user is not authorized to perform this operation.              |
| **404**          | `Category Not Found`       | One or more specified categories could not be found.               |


- **Query getUserCategories: [Category]**
Description: This endpoint retrieves all categories associated with the currently authenticated user. It is protected by authentication guards to ensure only authorized users can access their category data.

#### Feature
- **User-Specific Categories: Returns a list of categories tied to the current user.**
- **Role-Based Access: Protected by GqlAuthGuard to ensure only authenticated users can query their categories.**
- **Efficient Retrieval: Queries the database to fetch the user's associated categories.**

#### Request Body
```
getUserCategories { 
	getUserCategories 
		{
	 id 
	name 
	description 
	} 
}
```

| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **200** | `Success` | The request was successful, and the categories were retrieved. | |**401** | `Unauthorized` | The user is not authenticated or lacks the necessary permissions. | 
| **500** | `Internal Server Error` | An unexpected error occurred while processing the request. |

 ### *Users Module*
- **Mutation createCourse(input: CourseInput, groupIds: [String]): Course**

Description: endpoint allows creating a new course in the system. It supports adding associated skills, categories, and assigning coaches. The course can also include an optional video, and it ensures validation of all inputs. A default group must be created for each course.

#### Features

- **Create New Course: Enables creating a course with attributes like title, description, skills, categories, and coach assignments.**
- **Skills and Categories Handling**
    - **Automatically creates non-existing skills and categories.**
    - **Associates skills and categories with the new course.**
   
-**Video Integration: Adds a video to the course if provided.**
- **Order Index Management: Automatically assigns an orderIndex  based on the total number of existing courses.**
- **Default Group Requirement: Ensures that every course has a default group unless explicitly disabled.**
- **Rich Text Description: Initializes an editor-compatible rich text description for the course.**
- **Validation and Authorization: Includes validation for input fields and role-based authorization (SUPER_ADMIN and ADMIN roles only)**

```
mutation createCourse($input: CourseInput!, $groupIds: [String!]) {
  createCourse(input: $input, groupIds: $groupIds) {
    id
    title
    description
    video {
      id
      url
    }
    skills {
      id
      name
    }
    categories {
      id
      name
    }
    coach {
      id
      name
    }
    orderIndex
  }
}


```

| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **201**          | `Success`                    | The course was created successfully.                               |
| **400**          | `Default group is required`  | The default group for the course is missing.                       |
| **400**          | `Bad Request`                | Invalid or missing fields in the input payload.                    |
| **401**          | `Unauthorized`               | The user lacks permissions to create a course.                     |
| **500**          | `Internal Server Error`      | An unexpected error occurred during the course creation process.   |


-  **Mutation editCourse(id: String!, input: CourseInput, groupIds: [String]): Course**

Description: This endpoint allows editing an existing course in the system. It supports updating course details such as title, description, associated skills, categories, and coach assignments. Additionally, it performs validation and role-based access control.


#### Features


- **Edit Course Details:**
  - **Update course attributes, including title, description, skills, categories, and coaches.**
- **Rich Text Description: Converts the provided course description into a structured     editor-compatible format.**
- **Skills and Categories Management:**
  - **Adds non-existing skills and categories to the database before associating them with the course.**
- **Validation and Authorization:**
  - **Ensures that only valid data is processed.**
 - **Ensures that only valid data is processed.**
- **Restricted to users with roles SUPER_ADMIN and ADMIN.**
- **Error Handling: Handles scenarios where the course does not exist or invalid data is provided.**

```
mutation editCourse($id: String!, $input: CourseInput, $groupIds: [String!]) {
  editCourse(id: $id, input: $input, groupIds: $groupIds) {
    id
    title
    description
    video {
      id
      url
    }
    skills {
      id
      name
    }
    categories {
      id
      name
    }
    coach {
      id
      name
    }
    updatedAt
  }
}


```

| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **200**          | `Success`              | The course was successfully updated.                               |
| **400**          | `Bad Request`          | Invalid or missing fields in the input payload.                    |
| **401**          | `Unauthorized`         | The user lacks permissions to edit the course.                     |
| **404**          | `Course Not Found`     | No course was found with the provided ID.                          |
| **500**          | `Internal Server Error`| An unexpected error occurred during the course update process.     |


- **Mutation: coursePublished(courseId: [String], action: String): [Course]**
Description: This mutation toggles the publication state of a course. A course can be either published or unpublished, and its state is updated accordingly. 


#### Features
- **Toggle Publication State**:
  - **Publish**: Marks the course as published (`isPublished: true`), updates its state to `published`, and assigns it an index for discovery or sorting.
  - **Unpublish**: Marks the course as unpublished (`isPublished: false`) and updates its state to `unpublished`.
- **Index Management**: Automatically updates course indexes when a course is published.
- **Validation and Authorization**:
  - Only users with roles `SUPER_ADMIN` and `ADMIN` can access this mutation.
  - Ensures the course exists before updating its state.



#### Request Body

```graphql
mutation coursePublished($courseId: [String!]!, $action: String!) {
  coursePublished(courseId: $courseId, action: $action) {
    id
    title
    isPublished
    state
    index
    updatedAt
  }
```
| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **200**          | `Success`              | The course publication state was successfully updated.             |
| **400**          | `Bad Request`          | Invalid or missing fields in the input payload.                    |
| **401**          | `Unauthorized`         | The user lacks permissions to publish or unpublish the course.     |
| **404**          | `Course Not Found`     | No course was found with the provided ID.                          |
| **500**          | `Internal Server Error`| An unexpected error occurred during the update process.            |



-  **Mutation: purchaseFreeCourse(courseId: String): SuccessPurchase **

Description: This endpoint allows a user to enroll in a free course, provided they have an active subscription. It validates the user's subscription and adds the user to the course. 

#### Features
- **Free Course Enrollment: Enables users to enroll in a free course by providing the course ID.**
- **Subscription Validation: Ensures the user has an active subscription before enrolling.** 
- **Error Handling: Handles scenarios where the user does not have an active subscription or if an unexpected error occurs during the enrollment process.**

#### Request Body

 ```
graphql mutation purchaseFreeCourse($courseId: String!) {             
    purchaseFreeCourse(courseId: $courseId) {
   status 
    }
 }
 
 
```

| **Status Code** | **Message**       | **Description**                                                     |
|------------------|-------------------------|---------------------------------------------------------------------|
| **200** | `Success` | The course was successfully added to the user's profile. |
| **400** | `Bad Request` | The user does not have an active subscription or other invalid input. |
| **401** | `Unauthorized` | The user is not authenticated. | 
| **404** | `Course Not Found` | No course was found with the provided ID. |
| **500** | `Internal Server Error` | An unexpected error occurred while processing the request. |


- **trackProgress(courseId: string, lessonId: string, duration: number): string | Promise<string>**

Description: Tracks the user's progress for a specific lesson within a course. This mutation updates the watched duration of the lesson and calculates the overall progress of the course. 

#### Features
 - **Track Lesson Progress: Updates the user's watched duration for a specific lesson.** 
- **Course Completion: Automatically marks the course as complete if the total watched duration meets or exceeds the course's total duration.**
- **Validation: Ensures that the duration being tracked does not exceed the lesson's actual duration.**
- **Progress Updates: Handles scenarios where progress already exists or needs to be initialized for the first time.**
- **Error Handling: Logs errors and validates inputs such as course and lesson IDs.**



#### Request Body

```
trackProgress($courseId: String!, $lessonId: String!, $duration: Float!) {   
        trackProgress(courseId: $courseId, lessonId: $lessonId, duration: $duration) 
}
```
#### Detailed Workflow

1. **Course Duration Validation**:
   - Fetches the course's total duration and lesson details.
   - Ensures that the lesson exists before tracking progress.

2. **Track Progress**:
   - If the progress already exists and the new duration is greater than the previous, it updates the record.
   - If no progress exists, initializes a new progress record for the lesson.

3. **Increase Watched Duration**:
   - Updates the total duration watched by the user for the course.
   - Marks the course as completed if the total watched duration meets the course's required duration.

4. **Error Handling**:
   - Logs any errors that occur during the creation or update of progress records.
   - Returns appropriate error messages for invalid input or internal server issues.

| **Status Code** | **Message**            | **Description**                                                         |
|------------------|------------------------------|-------------------------------------------------------------------------|
| **200**          | `Success`                   | The progress was successfully tracked.                                 |
| **400**          | `Bad Request`               | Invalid input, such as missing or invalid course/lesson IDs or duration. |
| **404**          | `Course or Lesson Not Found`| No course or lesson was found with the provided IDs.                   |
| **500**          | `Internal Server Error`     | An unexpected error occurred while tracking progress.                  |


### Mutation: addModule(input: ModuleInput, courseData: CourseData): Module

**Description**: This endpoint allows administrators to add a new module to a course. It validates the input, assigns the appropriate order index, and associates the module with the provided course details.

---

#### Features
- **Add New Module**: Enables the creation of a new module with a description, order index, and other details.
- **Order Index Assignment**: Automatically assigns the correct order index based on existing modules.
- **Course Association**: Associates the module with a course if `courseData` contains a valid course ID and name.
- **Rich Text Description**: Initializes the module description in a structured format for future use.
- **Error Handling**: Handles scenarios such as missing required fields or unexpected database errors.

---

#### Request Body

```graphql
mutation addModule($input: ModuleInput!, $courseData: CourseData!) {
  addModule(input: $input, courseData: $courseData) {
    id
    name
    description
    orderndex
    courseId
  }
}
```

#### Status Codes and Error Messages

| **Status Code** | **Message**                | **Description**                                                   |
|------------------|----------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                       | The request was successful, and the module was created successfully. |
| **400**          | `Bad Request`                   | The request contains invalid or missing fields in the input.      |
| **401**          | `Unauthorized`                  | The user is not authorized to add a module due to lack of permissions. |
| **404**          | `Course Not Found`              | The provided course ID does not exist in the database.            |
| **500**          | `Internal Server Error`         | An unexpected error occurred during the module creation process.  |

- **Query: allModules(filter: CourseFilter, currentPage: Int, perPage: Int, fetchAll: Boolean): FetchModulesResponse**

Description: This endpoint retrieves a paginated list of modules based on the provided filter criteria. It supports flexible filtering, role-based access, and efficient pagination.

#### Features
- **Flexible Filtering**: Users can apply various filters using the `CourseFilter` parameter.
- **Pagination**: Supports pagination through `currentPage` and `perPage` arguments.
- **Fetch All Option**: Provides the ability to fetch all modules in one request with `fetchAll: true`.
- **Role-Based Access**: Access is restricted to users with roles: `SUPER_ADMIN`, `ADMIN`, `COACH`, and `STUDENT`.
- **Course Linking**: Automatically adjusts the `courseId` in the filter if the course has been purchased or has an original ID.

#### Request Body
```graphql
query allModules(
  $filter: CourseFilter, 
  $currentPage: Int, 
  $perPage: Int, 
  $fetchAll: Boolean
) {
  allModules(
    filter: $filter, 
    currentPage: $currentPage, 
    perPage: $perPage, 
    fetchAll: $fetchAll
  ) {
   currentPage,
  totalPages, 
  totalCount,
  data: [Module]
  }
}
```
| **Status Code** | **Message**             | **Description**                                                   |
|------------------|-------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                    | The modules were successfully retrieved.                         |
| **400**          | `Bad Request`                | Invalid input or an error occurred while processing the query.   |
| **401**          | `Unauthorized`               | The user does not have permission to access this endpoint.       |
| **404**          | `Course Not Found`           | The provided course ID does not exist or is invalid.             |
| **500**          | `Internal Server Error`      | An unexpected error occurred while processing the request.       |

- **Mutation: addModulesToCourse(courseData: CourseData, moduleIds: [String]): [Module]**

Description: This endpoint allows adding one or more modules to a specific course. It associates the provided modules with the given course.

#### Features
- **Course Association:** Adds multiple modules to the specified course.
- **Access Control:** Restricted to `SUPER_ADMIN` and `ADMIN` roles.
- **Error Handling:** Provides meaningful error messages for invalid inputs or server errors.

#### Request Body
```graphql
mutation addModulesToCourse($courseData: CourseData!, $moduleIds: [String!]!) {
  addModulesToCourse(courseData: $courseData, moduleIds: $moduleIds) {
Module
  }
}
```
| **Status Code** | **Message**             | **Description**                                                   |
|------------------|-------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                    | The modules were successfully added to the course.               |
| **400**          | `Bad Request`                | Invalid course data or module IDs provided.                      |
| **401**          | `Unauthorized`               | The user is not authorized to perform this action.               |
| **404**          | `Course or Module Not Found` | The course or one of the module IDs does not exist.              |
| **500**          | `Internal Server Error`      | An unexpected error occurred while processing the request.       |


### Subscriptions Module

### `Mutation: createSubscription(input: CreateSubscriptionInput!): Subscriptions`

**Description**: This endpoint allows a user to create a new subscription. It validates whether the user already has an active subscription, checks if the requested subscription product exists, and then creates the subscription. If the user already has a subscription, an error will be returned.

---

#### Features
- **User Subscription Validation**: Checks if the user already has an active or on-hold subscription before creating a new one.
- **Subscription Product Validation**: Ensures the provided subscription product ID exists in the system.
- **Subscription Creation**: Creates a new subscription and links it to the user's account.
- **Error Handling**: Provides meaningful error messages for issues such as existing subscriptions or non-existent products.

---

#### Request Body

```graphql
mutation createSubscription($input: CreateSubscriptionInput!) {
  createSubscription(input: $input) {
  Subscriptions
  }
}
```
#### Status Codes and Error Messages

| **Status Code** | **Message**                     | **Description**                                                   |
|------------------|---------------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                             | The subscription was successfully created.                        |
| **400**          | `User already has active subscription`| The user already has an active or on-hold subscription.           |
| **400**          | `Subscription product not found`      | The provided subscription product ID does not exist.             |
| **500**          | `Internal Server Error`               | An unexpected error occurred while creating the subscription.     |

### Mutation: `activateAdminSubscription(subscriptionId: String!, orderId: String!):Subscriptions`

#### **Description**:
This endpoint allows administrators (with `SUPER_ADMIN` or `ADMIN` roles) to activate a subscription. It validates the subscription and product, calculates the appropriate payment dates, and updates the subscription status to `SUBSCRIBED`.



#### **Features**
- **Subscription Validation**: Ensures the provided subscription exists and is active.
- **Product Validation**: Verifies that the associated product exists in the system.
- **Payment Date Calculation**: Automatically calculates `nextPaymentDate` and `lastPaymentDate` based on the subscription product's configuration.
- **Subscription Activation**: Updates the subscription status to `SUBSCRIBED` and links it to the provided order ID.
- **Role-Based Access**: Only accessible to users with the `SUPER_ADMIN` or `ADMIN` roles.



#### **Request Body**

```graphql
mutation activateAdminSubscription($subscriptionId: String!, $orderId: String!) {
  activateAdminSubscription(subscriptionId: $subscriptionId, orderId: $orderId) {
 Subscriptions
  }
}
```



| **Status Code** | **Message**                | **Description**                                                    |
|------------------|----------------------------------|--------------------------------------------------------------------|
| **200**          | `Success`                       | The request was successful, and the subscription was activated.    |
| **400**          | `Subscription not found`        | The provided subscription ID does not exist or is inactive.       |
| **400**          | `Subscription product not found`| The associated subscription product could not be found.           |
| **500**          | `Internal Server Error`         | An unexpected error occurred while activating the subscription.   |


### Mutation: cancelSubscription(id: String!): Subscriptions

**Description**: This endpoint allows a user to cancel their subscription. It first checks if the subscription exists and if it is on hold. If so, it updates the subscription status to "CANCELED" and clears the associated card token. The user’s payment details are also updated as needed.


#### Features
- **Subscription Cancellation**: Cancels the subscription by setting the status to "CANCELED" and nullifying the card token.
- **Status Check**: Validates if the subscription is on hold before canceling it.
- **User Update**: Updates user details like payment failure status after cancellation.
- **Error Handling**: Handles errors such as subscription not found or any internal server issues.



#### Request Body

```graphql
mutation cancelSubscription($id: String!) {
  cancelSubscription(id: $id) {
   Subscriptions
  }
}

```


### Mutation: updateSubscriptionNextPaymentDate(userId: String!, subscriptionId: String!): Subscriptions

**Description**: This endpoint updates the next payment date of a user's subscription. It first validates the existence of the user and subscription. If both are found, it calculates the next payment date based on the subscription product details and updates the subscription with the new payment information.



#### Features
- **Next Payment Date Calculation**: Calculates the next payment date based on the subscription product details and the user's payment history.
- **Subscription Update**: Updates the subscription with the new payment date and increments the payment count.
- **Error Handling**: Handles scenarios like missing user, subscription, or subscription product.
- **Logging**: Logs critical details like the subscription and next payment date for debugging.



#### Request Body

```graphql
mutation updateSubscriptionNextPaymentDate($userId: String!, $subscriptionId: String!) {
  updateSubscriptionNextPaymentDate(userId: $userId, subscriptionId: $subscriptionId) {
   Subscriptions
  }
}
```



| **Status Code** | **Message**                  | **Description**                                                   |
|------------------|------------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                         | The subscription's next payment date was successfully updated.    |
| **400**          | `Bad Request`                     | The request contains invalid parameters, or the subscription update failed. |
| **404**          | `User Not Found`                  | The user with the provided ID could not be found.                 |
| **404**          | `Subscription Not Found`          | The subscription with the provided ID could not be found.         |
| **404**          | `Subscription Product Not Found`  | The subscription product associated with the subscription is not found. |
| **500**          | `Internal Server Error`           | The next payment date could not be updated due to an internal error. |






### Query: getActiveSubscriptions: [Subscriptions]

**Description**: This query retrieves a list of active subscriptions, which includes subscriptions that are either "SUBSCRIBED" or "ON_HOLD" and have a `nextPaymentDate` that is less than or equal to the current date. The result excludes any subscriptions with a `null` `originalTransactionId`.

---

#### Features
- **Retrieve Active Subscriptions**: Filters subscriptions that are active (status: SUBSCRIBED or ON_HOLD) and have a valid `nextPaymentDate`.
- **Excludes Null Transaction IDs**: Only subscriptions with a non-null `originalTransactionId` are included.
- **Error Handling**: If there is an issue during the subscription retrieval process, it returns a `BAD_REQUEST` error.

---

#### Request Body

```graphql
query getActiveSubscriptions {
  getActiveSubscriptions {
   Subscriptions
  }
}

```



| **Status Code** | **Message**            | **Description**                                                   |
|------------------|------------------------------|-------------------------------------------------------------------|
| **200**          | `Success`                |The request was successful, and the active subscriptions were retrieved.. |
| **400**          | `Bad Request`                | The request could not be processed due to invalid parameters or a problem retrieving active subscriptions. |
| **500**          | `Internal Server Error`      | An unexpected error occurred while retrieving the active subscriptions. |



### API Documentation for Payment Microservice

#### Endpoint: POST/ ```subscribe```

##### Description:
This endpoint handles the subscription process for users. It validates the subscription product, ensures the user does not already have an active subscription, and facilitates the payment process, ultimately returning a redirection URL for payment completion.


#### Features
- **Subscription Validation: Ensures the provided subscription product exists and verifies the user doesn't already have an active subscription.**
- **Environment Configuration: Dynamically selects the environment (test or prod) based on the redirect_url.**
- **Order Creation: Creates a pending subscription and initiates an order for payment with multiple payment methods (card, Google Pay, Apple Pay).**
- **Error Handling: Provides meaningful error messages for issues like missing product, active subscriptions, or payment failures.**
- **Card Saving (Optional): Sends a request to save card details for future payments.**
#### Detailed Workflow

### Environment Setup:
- Determines the environment (`test` or `prod`) based on the `redirect_url`.
- Retrieves the Apollo client instance for the appropriate environment.

### Product and Subscription Validation:
- Fetches the subscription product using `getSubscriptionProduct()`.
- Checks if the user already has an active subscription using `getUserActiveSubscription()`.

### Pending Subscription Creation:
- Creates a pending subscription with status `PENDING` via `createPendingSubscription()`.

### Order Details Preparation:
- Constructs the order details object with:
  - `callback_url`
  - Payment methods (card, Google Pay, Apple Pay)
  - Product details (price, ID, etc.)

### Order Request:
- Sends the order request to the bank API using the `order()` function.
- Handles card-saving details if applicable.

### Redirection:
- Extracts the redirection URL from the bank API response.
- Responds with the redirection URL for the user to complete the payment.

### Error Handling:
- Returns appropriate errors for invalid products, active subscriptions, authentication failures, or order processing issues.

| **Status Code** | **Message**                  | **Description**                                               |
|------------------|------------------------------------|---------------------------------------------------------------|
| **200**          | `Success`                          | The subscription was successfully processed.                  |
| **400**          | `Subscription Product Not Found`   | The provided product ID does not exist.                       |
| **400**          | `User Already Has Active Subscription` | The user is already subscribed to a product.                |
| **401**          | `Bank Authentication Failed`       | Bank authentication failed.                                   |
| **500**          | `Order Product Failed`             | An internal server error occurred while processing the order. |




#### Endpoint: POST/ ```payment-status-test```

##### Description:
This endpoint handles the payment status process for users. It retrieves the order details, checks the payment status, and based on the result, either creates a transaction, activates or cancels a subscription, or handles recurrent payment. It returns appropriate responses for successful or failed payment processing.

#### Features
- **Payment Status Handling: Fetches the order details and payment status, then processes accordingly.**
- **Subscription Management: Activates or cancels subscriptions based on payment status.**
- **Transaction Creation: Creates a transaction record for both successful and failed payments.**
- **Recurrent Payment Handling: Updates the next payment date for recurrent payments.**
- **Error Handling: Provides meaningful error messages for issues like failed payments or missing order details.**

#### Detailed Workflow

### Payment Details Retrieval:
- Fetches the payment order details using `getOrderDetailsV2()` based on the `order_id` passed in the request.

### Subscription and Transaction Processing:
- If the payment status is "rejected" and the payment type is `RECURRENT`, a new transaction is created, and the subscription is either canceled or kept pending.
- For non-recurrent payments, the subscription is activated, and a transaction is created with the status `PERFORMED`.
- For recurrent payments, updates the next payment date and creates a transaction.

### Redirection:
- Responds with the appropriate status (`payment failed`, `subscription canceled`, `subscription activated`, etc.).

### Error Handling:
- Returns appropriate errors for invalid orders, active subscriptions, or payment failures.

| **Status Code** | **Message**                  | **Description**                                               |
|------------------|------------------------------------|---------------------------------------------------------------|
| **200**          | `payment failed`                   | The payment failed, and the subscription remains in pending state. |
| **200**          | `subscription canceled`            | The subscription was canceled due to payment failure.         |
| **200**          | `subscription activated`           | The subscription was successfully activated after payment.    |
| **200**          | `subscription reactivated`         | The subscription was reactivated for recurrent payments.      |
| **400**          | `Invalid Order ID`                 | The provided order ID is invalid or not found.                |
| **500**          | `Order Processing Error`           | An internal server error occurred while processing the payment or subscription. |
### Mutation: sendSubscriptionPaymentDetails(userId: String!, subscriptionId: String!): SuccessResponse

**Description**: This mutation retrieves the subscription details and sends the payment information via email to the user. It formats the currency based on the subscription's product and prepares the payment details for sending. If successful, it returns a success status.

#### Features
- **Retrieve Subscription Details**: Fetches the subscription details using the `subscriptionId` and checks if the subscription exists.
- **Currency Formatting**: Formats the currency into the appropriate Georgian Lari (GEL), U.S. Dollar (USD), or Euro (EUR).
- **Send Payment Details**: Sends the payment information via email (though the email service code is commented out in the provided code).
- **Error Handling**: If the subscription or product is not found, or if an error occurs during processing, it returns a `BAD_REQUEST` error.


#### Request Body

```graphql
mutation sendSubscriptionPaymentDetails($userId: String!, $subscriptionId: String!) {
  sendSubscriptionPaymentDetails(userId: $userId, subscriptionId: $subscriptionId) {
    status
  }
}

```

| **Status Code** | **Error Message**                      | **Description**                                                    |
|------------------|----------------------------------------|--------------------------------------------------------------------|
| **200**          | `Success`                             | The payment details were successfully processed and sent.          |
| **400**          | `Bad Request`                         | The subscription or product ID was not found.                      |
| **500**          | `Internal Server Error`               | An unexpected error occurred during the payment details process.   |



### Transactions Module

#### Mutation: createSubscriptionTransaction(input: CreateTransactionInput!): Transaction

**Description**: This mutation handles the creation of a new subscription transaction. It processes the payment status and updates both the user’s points and subscription status based on the transaction outcome. If successful, it sends an activation email for new orders and updates the subscription status for recurring payments. Additionally, it calculates the points based on the subscription’s occurrence number and sends payment details to the payments microservice.


#### Features
- **Transaction Creation**:Creates a new transaction entry with details such as order ID, user ID, payment status, and other relevant information.
- **User Points Update**: If the payment is successful, the user’s points are updated based on the subscription occurrence number (e.g., 1 month, 6 months, or 12 months)..
- **Subscription Status Management**: Updates the subscription status depending on the payment type and result (success or error).
- **Email Notification**:  Sends an activation email for new orders.
- **Sending Payment Details**:  Sends payment details to the payments microservice to inform it about the transaction outcome.
- **Error Handling**:  Sends payment details to the payments microservice to inform it about the transaction outcome.

#### Request Body

```
mutation createSubscriptionTransaction($input: CreateTransactionInput!) {
  createSubscriptionTransaction(input: $input) {
  Transaction
  }
}
```
| **Status Code** | **Error Message**             | **Description**                                                   |
|------------------|-------------------------------|-------------------------------------------------------------------|
| **400**          | `Bad Request`                 | Missing required fields such as `order_id`, `shop_order_id`, or `userId`. |
| **404**          | `Subscription Not Found`      | No subscription found for the provided `subscriptionId`.          |
| **404**          | `User Not Found`              | No user found for the provided `userId`.                          |
| **500**          | `Internal Server Error`       | An unexpected error occurred during the transaction creation process. |

#### Query: `getTransactions(currentPage: Int!, perPage: Int!, fetchAll: Boolean!): PaginatedTransactions`

#### Description
This query retrieves a list of transactions, with pagination support. You can specify the current page and the number of items per page. If the `fetchAll` parameter is set to true, it will return up to 1000 records. The query will return paginated transaction data, including the total number of transactions available.


#### Features
- **Pagination**: Supports pagination with `currentPage` and `perPage` arguments. If `fetchAll` is true, it will return all transactions up to a limit of 1000.
- **Fetch All Transactions**: If `fetchAll` is set to `true`, the query will return all records, up to a maximum of 1000.
- **Transaction Data**: Retrieves a list of transaction records, including details like transaction ID, status, payment method, and more.


#### Request Body

```graphql
query getTransactions($currentPage: Int!, $perPage: Int!, $fetchAll: Boolean!) {
  getTransactions(currentPage: $currentPage, perPage: $perPage, fetchAll: $fetchAll) {
 currentPage,
  totalPages,
  totalCount,
  data: [Transaction]
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `success`             |trasaction retrieved successfuly |
| **400**          | `Bad Request`             | Invalid parameters passed, such as missing or invalid `currentPage` or `perPage`. |
| **500**          | `Internal Server Error`   | An error occurred while processing the request.                |

#### Query: `getTransactions(currentPage: Int!, perPage: Int!, fetchAll: Boolean!): TransactionsResponse`

#### Description
This query retrieves a list of transactions with pagination support. You can specify the current page and the number of items per page. If `fetchAll` is set to `true`, it will return up to 1000 transactions. The query returns a paginated list of transactions along with the total count of transactions.



#### Features
- **Pagination Support**: The query allows pagination through the `currentPage` and `perPage` arguments.
- **Fetch All Transactions**: When `fetchAll` is set to `true`, up to 1000 transactions are returned.
- **Transaction Data**: Retrieves detailed information for each transaction including order ID, user ID, status, amount, payment method, and more.
- **Total Count**: Includes the total number of transactions available for paginated results.



#### Request Body

```graphql
query getTransactions($currentPage: Int!, $perPage: Int!, $fetchAll: Boolean!) {
  getTransactions(currentPage: $currentPage, perPage: $perPage, fetchAll: $fetchAll) {
    currentPage,
  totalPages, 
  totalCount,
  data{
	...TransactionFragment
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `success`             |trasactions retrieved successfully |
| **400**          | `Bad Request`             | Invalid parameters passed, such as missing or invalid `currentPage` or `perPage`. |
| **500**          | `Internal Server Error`   | An error occurred while processing the request.  |



### Topics Module

#### Mutation: `addTopic(moduleId: String, input: TopicInput): Module`

#### Description
The addTopic mutation allows adding a new topic to a specified module. It creates a topic within the system, associates it with the given module, and updates the module with the newly created topic.
This mutation is restricted to users with specific roles, such as `SUPER_ADMIN` and `ADMIN`.

#### Features
- **Role-Based Access Control**: Only users with the SUPER_ADMIN or ADMIN role can execute this mutation.
- **Validation**: Ensures input conforms to defined constraints using a validation pipe.
- **Create Topic**: Adds a new topic to a module using the provided moduleId and TopicInput.
- **Associate Topic**: Links the created topic with the specified module.


#### Request Body

```graphql
mutation addTopic($moduleId: String!, $input: TopicInput!) {
  addTopic(moduleId: $moduleId, input: $input) {
   ...moduleFragment
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `success`             |trasactions retrieved successfully |
| **400**          | `Bad Request`             | Invalid parameters passed, such as missing or invalid `currentPage` or `perPage`. |
| **500**          | `Internal Server Error`   | An error occurred while processing the request.  |



#### Query: `topic(topicId: String!): Topic`

#### Description
The topic query retrieves the details of a specific topic by its unique ID. It includes information about the topic and its associated lessons, sorted in ascending order based on their orderIndex.
This query is accessible to users with roles such as SUPER_ADMIN, ADMIN, COACH, and STUDENT.


#### Features
- **Role-Based Access Control**: Restricts access to specific roles (SUPER_ADMIN, ADMIN, COACH, STUDENT).
- **Retrieve Topic Details**: Fetches a topic by its ID from the database.
- **Lesson Sorting**: Ensures lessons within the topic are returned in ascending order of their orderIndex.
- **Error Handling**: Returns meaningful error messages when the topic is not found or other issues occur.

#### Request Body

```graphql
query getTopic($topicId: String!) {
  topic(topicId: $topicId) {
    topic
  }
}

```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `success`             |topic retrieved successfully |
| **403**          | `Unauthorized`             | The user does not have the required permissions.|
| **500**          | `Internal Server Error`   | An unexpected error occurred during the operation.|


#### Mutation: `deleteTopics(moduleId: String, topicIds: [String]): Module`

#### Description
The deleteTopics mutation allows authorized users to delete multiple topics from a specific module by providing the moduleId and a list of topicIds. It returns the updated module after the topics have been deleted.
This mutation is accessible only to users with the roles SUPER_ADMIN or ADMIN.


#### Features
- **Batch Deletion**: Removes multiple topics at once based on the provided topicIds.
- **RBAC Enforcement**: Ensures only users with roles like SUPER_ADMIN and ADMIN can perform this action.
- **Modular Integrity**: Operates within the context of a specific module, identified by its moduleId.


#### Request Body

```graphql
mutation deleteTopics($moduleId: String!, $topicIds: [String]!) {
  deleteTopics(moduleId: $moduleId, topicIds: $topicIds) {
   module
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `success`             |topic deleted successfully |
| **400**          | `Topic IDs invalid	`             | One or more topicIds are not valid.|
| **403**          | `Unauthorized`   | The user does not have the required permissions.|



### Lessons Module

#### Mutation: `createLesson(topicId: String, input: LessonInput): Topic`

#### Description
The createLesson mutation allows authorized users to add a new lesson to a specific topic by providing a topicId and lesson details through the input object. The response includes the updated topic containing the newly created lesson.
This mutation is accessible to users with the SUPER_ADMIN or ADMIN roles.


#### Features
- **Lesson Creation**: Adds a new lesson to the specified topic.
- **Rich Text Formatting**: Converts the description and transcript fields into a JSON structure compatible with the Draft.js format.
- **Video Association**: Associates a video with the lesson if provided.
- **RBAC Enforcement**: Restricts access to authorized roles only.


#### Request Body

```graphql
mutation createLesson($topicId: String!, $input: LessonInput!) {
  createLesson(topicId: $topicId, input: $input) {
	topic
  }
}

```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Lesson created successfully |
| **400**          | `Topic not found`         | The specified topicId does not exist.|
| **400**          | `Invalid input data`      | The provided lesson details are invalid or incomplete.|
| **403**          | `Unauthorized`   | The user does not have the required permissions.|


#### Query: `getLessons(topicId: String): [Lesson]`

#### Description
The getLessons query retrieves all lessons associated with a specific topic. This endpoint supports role-based access control (RBAC) and allows authorized users to fetch lesson details by providing the topicId.
This query is accessible to users with the roles: SUPER_ADMIN, ADMIN, COACH, and STUDENT.

#### Features
- **Lesson Retrieval:**: Fetches all lessons linked to the specified topic.
- **RBAC Enforcement**:  Ensures that only authorized users can access this endpoint.

#### Request Body
```graphql
query getLessons($topicId: String!) {
  getLessons(topicId: $topicId) {
 lesson
}
```


| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Lessons retraved successfully |
| **400**          | `Topic not found`         | The specified topicId does not exist.|
| **403**          | `Unauthorized`   | The user does not have the required permissions.|


#### Query: `getLessonById(lessonId: String, courseId: String): Lesson`

#### Description
The getLessonById query retrieves the details of a specific lesson based on its lessonId and optionally filters by courseId. This endpoint is accessible to users with the roles SUPER_ADMIN, ADMIN, COACH, and STUDENT.

#### Features
- **Lesson Retrieval by ID**: Fetches a single lesson using its unique identifier.
- **Course Association**:  Optionally checks the lesson's association with the specified courseId.
- **RBAC Enforcement:**: Ensures only authorized users can access lesson details.

#### Request Body
```graphql
query getLessonById($lessonId: String!, $courseId: String) {
  getLessonById(lessonId: $lessonId, courseId: $courseId) {
   	lesson
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Lessons retraved successfully |
| **400**          | `Topic not found`         | The specified topicId does not exist.|
| **403**          | `Unauthorized`   | The user does not have the required permissions.|


#### Mutation: `editLesson(topicId: String, lessonId: String, input: LessonInput): Lesson`

#### Description
The editLesson mutation updates the details of a specific lesson based on its lessonId within the context of the provided topicId. It allows modification of various lesson attributes and ensures that only authorized users can perform this action.

#### Features
- **Update Lesson Details**: Modify lesson attributes such as name, description, transcript, or video details.
- **RBAC Enforcement**: Restricts access to SUPER_ADMIN and ADMIN roles.
- **Partial Updates**:Accepts partial data for updating only the necessary fields of a lesson.

#### Request Body
```graphql
mutation editLesson($topicId: String!, $lessonId: String!, $input: LessonInput!) {
  editLesson(topicId: $topicId, lessonId: $lessonId, input: $input) {
  	lesson
  }
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Lessons edited successfully |
| **404**          | `Lesson not found`         | The specified lessonId does not exist.|
| **403**          | `Unauthorized`   | The user does not have the required permissions.|


#### Mutation: `changeLessonsOrder(draggedId: String!, droppedId: String!): String`

#### Description
The changeLessonsOrder mutation changes the order of lessons within a specific topic. The mutation moves a lesson (draggedId) to a new position (droppedId). It allows users with SUPER_ADMIN or ADMIN roles to re-arrange lessons by adjusting their orderIndex values.


#### Features
- **Reorder Lessons**: Updates the order of lessons within a specific topic.
- **RBAC Enforcement**: Only accessible by users with SUPER_ADMIN or ADMIN roles.
- **Partial Updates**: Updates all lessons' orderIndex values in parallel to maintain the new order.

#### Request Body
```graphql
mutation changeLessonsOrder($draggedId: String!, $droppedId: String!) {
  changeLessonsOrder(draggedId: $draggedId, droppedId: $droppedId)
}
```

| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Lessons order edited successfully |
| **400**          | `Dragged lesson or topic not found`         |The dragged lesson or its associated topic does not exist.|
| **400**          | `Lessons not found in the topic`         |Either the dragged or dropped lesson could not be found in the topic.|


### Video Module

#### REST Endpoint: ` POST  videos/convert`

#### Description
This endpoint triggers a video conversion process by sending a request to an external video converter service. It requires the videoKey as part of the request body, which represents the name or identifier of the video to be converted.


#### Features
- **JWT Authentication**: Ensures that only authenticated users can trigger the video conversion process.
- **Validation**: The request body is validated using ValidationPipe to ensure proper data structure.


| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |Video converted successfully |
| **400**          | `Video file not found`         |The video specified by videoKey could not be found.|
| **401**          | `Unauthorized`         |The request did not include a valid JWT token.|


#### REST Endpoint: ` POST videos/conversion-success`

#### Description
This endpoint updates the status of a video's conversion once the conversion process is successful. It requires the videoKey and duration as part of the request body. The server then updates the video details, marking it as converted and possibly updating the video's duration if it's a lesson video.

#### Features
- **Validation**: The request body is validated using ValidationPipe to ensure proper data structure.
- **Conversion Status Update**: Marks the video's links as converted and updates the video duration if it's related to a lesson.
- **Conversion Status Update**: Marks the video's links as converted and updates the video duration if it's related to a lesson.


| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             |<video.key> |
| **400**          | `Video not found`         |The video specified by videoKey could not be found.|

#### REST Endpoint: ` POST videos/check-video-conversion-status`

#### Description
This endpoint checks the conversion status of a video by its link. It verifies whether all associated video links have been converted successfully. If the video is not found, it returns an error message.

#### Features
- **Authentication**: The request is protected by JWT authentication.
- **Validation**: The request body is validated to ensure it contains the correct videoLink.
- **Conversion Status Check**: Checks if all links associated with the video have been converted.


| **Status Code** | **Error Message**         | **Description**                                                 |
|------------------|---------------------------|-----------------------------------------------------------------|
| **200**          | `Success`             | <Boolean> |
| **400**          | `Video not found`         |The video specified by videoKey could not be found.|


### 8. Glossary
 
 #### Type User

```
{
id: ID! 
email: String 
firstName: String 
lastName: String
lastLoginDate: Date
isSuperAdmin: Boolean
isCompanyAdmin: Boolean
isIndividual: Boolean 
isQualified: Boolean 
isInternalCoach: Boolean
 isRegistered: Boolean
 becameCoachDate: DateTime
 coachMadeBy: ObjectId
 role: String
 isFirstLogin: Boolean
 biography: String
 note: String
 phone: String
 birthDate: String 
age: Int
 gender: String 
companyId: Company 
status: String 
createDate: Date 
modifiedDate: DateTime 
deletedDate: DateTime 
favoriteCourses: [Course] 
enrolledCourseScores: [EnrolledCourseScore] 
trainingPlan: [TrainingPlanItem] 
fullName: String
 avatar: String 
group: [UserGroup] 
requestPasswordChange: Boolean 
settings: UserSettings
courses: [UserCourses] 
jobTitle: String
location: String
 totalStudents: Int 
progress: Int 
phoneFields: PhoneFields 
education: [Education]
 experience: [Experience] 
averageRating: Int 
sharedCourses: [Course]
notes: [Note] 
subscription: Subscriptions 
canceledSub: Subscriptions 
points: Int 
firstFailedPayment: Date 
mobileNumberVerified: Boolean
 mobileNumber: String
 }
```
#### Type SignUpInput

```
{ 
email: string; 
password: string;
firstName: string,
lastName: string
}
```

#### Type LoginInput 
````
{ 
email: String! 
password: String!
 deviceId: String 
deviceType: String
 }
````


#### Type AppleAuth 
````
 {
 authorizationCode: String 
email: String 
fullName: FullName 
identityToken: String 
deviceId: String 
deviceType: String 
}
````

#### Type GoogleAuthInput 
````
{
 idToken: String! 
scopes: [String] 
user: GoogleUser 
deviceType: String 
deviceId: String 
}
````

####  Type FetchUsersResponse
```
{
currentPage: Int 
totalPages: Int
 totalCount: Int 
data: [User]
}
```

#### Type UserFilter 
```
 {
 firstName: FilterableField 
lastName: FilterableField
 createDate: FilterableField 
course: FilterableFieldArray 
status: FilterableField 
search: FilterableField
 note: FilterableField 
location: FilterableField 
}
```


#### Type FilterableField 
```
{
type: string 
value: string | [string] 
fields?: string[]
}
```

#### Type FilterableFieldArray

```
{
type: string 
value: string | [string] 
}
```
#### Type Course

```
{
  skills: [SkillInput]
avatar: AttachmentInput
certificateIncluded: Boolean
certificate: AttachmentInput
price: Float
currency: String
companyId: String
video: VideoInput
groups: [UserGroupInput]
subtitle: AttachmentInput
finished: Float
categories: [CourseCategoryInput]
contentLocked: Boolean
needDefaultGroup: Boolean
defaultGroupAdmin: String
index: Int
}
```
