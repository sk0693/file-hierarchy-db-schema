# Introduction

This repository is for creating the DB schema for handling the Folders and Files hierarchy approach and applying the CRUD operations smoothly. I have used MongoDB database for this, in which have I made parallel data document structure.

# Requirements

For this we need to maintain two collections named `user` collection and `nodes-library` collections. It can be any of the name of your choice.

#### User

```bash
`user` collection stores the details about the user
```

```javascript
{
    _id: ObjectID;
    email: String;
    name: String;
    rootFolderId: String;
    createdAt: Date;
    updatedAt: Date;
    ...
}
```

- `['_id']` - The unique id of any document. In `MongoDB` when the document is created, the ObjectId will automatically generate as the primary id of the document. **Required**
- `['email']` - The user email id.**Optional**.
- `['name']` - The user's name. **Optional**.
- `['type']` - Type of the document for handling the operations. Its either `file` or `folder`. **Required**
- `['rootFolderId']` - This key will contain the id of the Root(Home) directory, which will be created when the new `user` document created. **Required**.
- `['createdAt']` - Created at key will denote to the timestamp, when the user doc has created. **Required**.
- `['updatedAt']` - It will denote the updation timestamp of the document. **Required**.

and so on...

#### nodes-library

```bash
`nodes-library` collection stores all documents about the `files` and the `folders`.
```

This collection will hold both types of `nodes` either `file` or `folder`. You may maintain this structure using different collections files and folder structure respectively.

Although the documents of this collection will seprated by a key called `type`.

```javascript
{
    _id: ObjectID;
    title: String;
    description: String;
    type: String; // "file" or "folder"
    parentId: String;
    rootNode: Boolean;
    ancestors: Array;
    meta: {
        file_url: String;
        filePath: String;
        fileName: String;
        size: Number;
        originalName: String;
        mimeType: String
    };
    createdAt: Timestamp;
    updatedAt: Timestamp;
    ...
}
```

- `['_id']` - The unique primary Id **Required**
- `['title']` - The title of the document by is given by the user to view on the screen after creation. **Optional**.
- `['description']` - The description of the file or folder. **Optional**.
- `['parentId']` - The folder Id in which the file(s) will be uploaded. The current node will be called `child` node of the parent node which id is this. **Required**.
- `['rootNode']` - This will denote to get weather the the created doc is `Root` doc or not (which will be created on the creation of the `user`) **Required**
- `['ancestors']` - The array of the all parents `ids` till the `[Root | Home]` folder. **Optional**.
- `['meta']` - The object which holds the meta details about the file or folders.**Optional**.
- `['createdAt']` - Created at key will denote to the timestamp, when the user doc has created. **Required**.
- `['updatedAt']` - It will denote the updation timestamp of the document. **Required**.

and so on...

# CRUD operations

Some db operations I have mentioned below to perform queries to achieve this functionality.

### (C)reating the folder

Whenever the request come to create a `folder` with some required params.

```http
GET /v1/library/createFolder?path=/root/folderA/folderB&title=Swimming Images
```

Here path contains the string with `/` seprated and using this we will create a folder inside the `folderB` folder. `root` folder already created on the creation of the `user`. And I am assuming `folderA` and `folderB` is already created with this approach.

`root` --> `folderA` --> `folderB`

Now the query for creating the folder would be:

```javascript
{
  db.getCollection("nodes-library").insertOne({
    title: "Swimming Images",
    description: "Folder for all swimming images",
    parentId: "folderB",
    ancestors: ["root", "folderA", "folderB"],
  });
}
```

##### Output Example for Folder

```javascript
{
    _id : ObjectId("5eb96d645484db3785a5a81c"),
    title : "Swimming Images",
    description : "Folder for all swimming images",
    parentId: "folderB",
    ancestors: [
        "root",
        "folderA",
        "folderB"
    ],
    createdAt : ISODate("2020-05-12T15:21:08.260Z"),
    updatedAt : ISODate("2020-05-12T15:21:08.260Z"),
}
```

In this example I have created a folder titled `Swimming Images` has created inside the folder called `folderB`. So the `parentId` of this folder would be **folderB**.

The `ancestors` Array will contain the all parent ids from `root` to `folderB`. So in this case `root`, `folderA`, `folderB` will be the part of the ancestors array.

the path of this folder would be:

```bash
/root/folderA/folderB/Swimming Images
```

I am asuming the keys by random words. In real the value would be the actual ids.

### (C)reating (uploading) the file

Whenever the request come to create a `file` with some required params.

```http
POST /v1/library/uploadFile
```

| Parameter     | Type     | Description                                              |
| :------------ | :------- | :------------------------------------------------------- |
| `file`        | `File`   | **Required**.                                            |
| `path`        | `String` | **Required**. i.e. /root/folderA/folderB/Swimming Images |
| `title`       | `String` | **Optional**. i.e. Learn Swimming                        |
| `description` | `String` | **Optional**. i.e. My mp4 video                          |

These all the params are required to upload the file.

```javascript
{
  db.getCollection("nodes-library").insertOne({
    title: "Learn Swimming",
    description: "My mp4 video",
    parentId: "Swimming Images",
    ancestors: ["root", "folderA", "folderB", "Swimming Images"],
    meta: {
      isCompressed: false,
      filePath: "/uploads/",
      fileName: "5bcfed38-3c05-47d3-a838-49be5d7dd09d.mp4",
      fileUrl: "/uploads/5bcfed38-3c05-47d3-a838-49be5d7dd09d.mp4",
      size: 44745607,
      originalName: "output_custom5.mp4",
      mimeType: "video/mp4",
    },
  });
}
```

After executing the command the response would be:

##### Example for File

```javascript
{
    _id : ObjectId("5eb96d645484db3785a5a81c"),
    title : "Learn Swimming",
    description : "My mp4 video",
    parentId: "folderC",
    ancestors: [
        "root",
        "folderA",
        "folderB",
        "folderC",
    ],
    meta : {
        isCompressed : false,
        filePath : "/uploads/",
        fileName : "5bcfed38-3c05-47d3-a838-49be5d7dd09d.mp4",
        fileUrl : "/uploads/5bcfed38-3c05-47d3-a838-49be5d7dd09d.mp4",
        size : 44745607,
        originalName : "output_custom5.mp4",
        mimeType : "video/mp4",
    },
    createdAt : ISODate("2020-05-12T15:21:08.260Z"),
    updatedAt : ISODate("2020-05-12T15:21:08.260Z"),
}
```

Here `parentId` and `ancestors` are used for reading, updating and the deleting document(s).

### (R)eading the files or folders

So the read operation for getting the all `files` and `folders` which all residing in a particular directory. So for getting this, `parentId` will be used (All nodes which has the given `parentId`)

```javascript
{
  db.getCollection("nodes-library").find({
    parentId: `[parentId]`,
  });
}
```

### (U)pdating any files or folders

We are using parallel data structure approach in which updation is very easy task. So when we have to update the doc for `file` or `folder`, we have a unique identifier `_id` for every doc.

```javascript
{
  db.getCollection("nodes-library").update(
    {
      _id: `[_id]`,
    },
    {
      $set: {
        title: "Swimming Lectures",
      },
    }
  );
}
```

For `Copy/Paste` we just need to update the `ancestors` Array and the `parentId`. It will simply cut the link from the previous node and will make a new connection.

### (D)eleting any files or folders

For achieving delete functionality two different operation will have to be performed respective to files and folder.

#### To delete any folder

For this we have to delete all of its childs folders and files. So using `ancestors` array this can be done easily.

For eg: Lets assume this folder hierarchy:

```bash
Root(HOME)
├── FolderA
│   ├── FolderB
│   │   ├── file1.jpg
│   │   ├── FolderB1
│   │   │       ├──FolderB12
│   │   │       └──Fileb123
│   │   ├── FolderB2
│   │   └── file1.jpg
│   └── FolderA2
│── FolderC
│── FolderD
└── FolderE
```

Here I am going to delete the `FolderB` from the above hierarchy. So to do this we have to delete all the nodes which having `FolderB` id in the `ancestors` Array.

```javascript
{
  db.getCollection("nodes-library").remove({
    ancestors: FolderB["_id"],
  });
}
```

After delete the `folderB` the remaining nodes hierarchy would be:

```bash
Root(HOME)
├── FolderA
│   └── FolderA2
│── FolderC
│── FolderD
└── FolderE
```

#### To delete any file

```javascript
{
  db.getCollection("nodes-library").remove({
    _id: Fle["_id"],
  });
}
```

So at the end all operations can be done easily on this `file hierarchy` using `parallel data structure` pattern.

## Authors

- **Sourabh Khurana**

* [GitHub](https://github.com/sk0693)
* [LinkedIn](https://linkedin.com/sk0693)
* [Portfolio](https://sourabhkhurana.com/resume.html)
