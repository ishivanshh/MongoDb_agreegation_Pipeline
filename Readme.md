# MongoDB Agreegation Pipeline.

- https://gist.github.com/ishivanshh/b560bf0bb0a7ebdc03cb1566ff3b0613

1. Download vscode extension : MongoDb for VS code
2. add string of your mongodb atlas with password 
3. use github repo to create three different clusters,
- books
- users
- authors
4. add json code to these clusters with 
'insert document' and paste code here 
5. check into atlas 
- go to agreegation columm 
> sample record
```
{
      "index": NumberInt(0),
      "name": "Aurelia Gonzales",
      "isActive": false,
      "registered": ISODate("2015-02-11T04:22:39+0000"),
      "age": NumberInt(20),
      "gender": "female",
      "eyeColor": "green",
      "favoriteFruit": "banana",
      "company": {
        "title": "YURTURE",
        "email": "aureliagonzales@yurture.com",
        "phone": "+1 (940) 501-3963",
        "location": {
          "country": "USA",
          "address": "694 Hewes Street"
        }
```
## Now create a agreegation pipeline.
```
[
  {
    $match: {
      isActive: true
    }
  },
  {
    $count: "activeUsers"
  }
]
```
always enclose in array [ {}, {}....]
- {stage-1}, {stage-2}....
- all stages are connected to each other 
1. How many active users 
- $match :- matches the isActive : true condition in all the records,
- $count :- now in all the matches records it will count how many are isActivr true with name "activeUsers"

2. what is the average age of all the users?
```
[
  {
    $group : {
      _id : "null",
      averageAge : {
        $avg : "$age"
      }
    }
  }
]
```
> $group => it group all the samples based on null(everything)
- then used $avg => for age section in sample records.

3. list the top 5 most common favorite fruits among users.

```
[
  {
    $group: {
      _id: "$favoriteFruit",
      countNoOfFruits : {
        $sum : 1
      }
    }
  },
  {
    $sort : {
      countNoOfFruits : -1
    }
  },
  {
    $limit: 2,
  }
]
```
- group by favoriteFruit and used $sum which will return the result how many times a particular fruits appears, 1+1+1+1... (accumulator)
- then sort by ascending order -1, for descending order 1
- limit => return the result top 2 

4. How many male and female are there ?
```
[
  {
    $group: {
      _id: "$gender",
      countNoEye : {
        $sum : 1
      }
    }
  }
]
```

5. which country has the highest number of registered users?

```
[
  {
    $group: {
      _id : "$company.location.country",
      userCount : {
        $sum : 1
      }
    }
  },
  {
    $sort: {
      userCount: -1
    }
  },
  {
    $limit: 5
  }
]
```
5. list all the eye colors 
```
[
  {
    $group: {
      _id: "$eyeColor",
      countNoEye : {
        $sum : 1
      }
    }
  }
]
```

6. what is the average number of tags per user ?

```
[
  {
    $unwind: {
      path: "$tags"
    }
  },
  {
    $group: {
      _id: "$_id",
      numberOfTags : {
        $sum : 1
      }
    }
  },
  {
    $group: {
      _id: "null",
      averageUserPerTags : {
        $avg : "$numberOfTags"
      }
    }
  }
]
```

> one more way to solve this,
```
[
  {
    $addFields: {
      NoOfTagForUser:{
        $size : {$ifNull : ["$tags", [ ] ]}
      }
    }
  },
  {
    $group: {
      _id : null,
      avergaeNoOfTags : {
        $avg : "$NoOfTagForUser"
      }
    }
  }
]
```
> $addFields => this field add the tag in the json 
- size => it represents no.of the repeated tag which is mentioned, 
- {$ifNull : ["$tags], []} => means if size is not possible return empty array or if possible then shows tags

7. how many enim are there in json objects 

[
  {
    $match: {
      tags : "enim"
    }
  },
  {
    $count: "tags"
  }
]
8. what are the age and name of the user who are inactive and have velit in there name ?
```
[
  {
    $match: {
      isActive : false,
      tags : "velit"
    }
  },
  {
    $project: {
      name : 1,
      age : 1
    }
  }
]
```
9. show most recent registered users?
```
[
  {
    $sort: {
      registered: -1
    }
  },
  {
    $limit: 4
  },
  {
    $project: {
      name : 1,
      favoriteFruit : 1,
      isActive : 1
    }
  }
]
```