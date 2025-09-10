---
title: "How to setup PostgreSQL for Dotnet webapi's using Entity Framework Core."
datePublished: Wed Sep 10 2025 03:35:16 GMT+0000 (Coordinated Universal Time)
cuid: cmfdffd54000202l7hpj761sn
slug: how-to-setup-postgresql-for-dotnet-webapis-using-entity-framework-core
cover: https://cdn.hashnode.com/res/hashnode/image/stock/unsplash/fPkvU7RDmCo/upload/90b61a2d34540ab635e6082c94ba8beb.jpeg
tags: postgresql, orm, dotnet, dotnetcore, pgsql, entityframeworkcore, npgsql

---

here our goal is two fold:

1. implement database functionalities using entity framework core.
    
2. create an endpoint and do crud on it
    

NOTE: *You can clone the repo with the code in the article at:* [*https://github.com/Umer-Islam/dotnet\_auth\_with\_db\_referenceRepo*](https://github.com/Umer-Islam/dotnet_auth_with_db_referenceRepo)

## Implementing Database using Entity Framework Core

### i. add the required packages

in this tutorial we will be using PostgresSQL, add the following packages:

```plaintext
dotnet add package Microsoft.EntityFrameworkCore.Tools     --version   9.0.9    
dotnet add package Npgsql.EntityFrameworkCore.PostgreSQL      --version  9.0.4   
```

### ii. create DbContext configuration file

this file will create the actual tables for us in the database, using our models(more on models below)

create a Data folder in root and within that create a class named AppDbContext with the following code:

```plaintext

using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.EntityFrameworkCore;

namespace dotnet_auth_with_db.Data
{
    public class AppDbContext(DbContextOptions<AppDbContext> options) : DbContext(options)
    {
       public DbSet<UserModel> Users{ get; set; } 
    }
}
```

this file will give you an error that UserModel doesn’t exist so this is the next thing that we will do.

create a models folder in root and within that a file named UserModels.cs, add the following code:

```plaintext
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace dotnet_auth_with_db.Models
{
    public class UserModel
    {
        public Guid Id { get; set; }
        public string Email { get; set; } = string.Empty;
        public string Password { get; set; } = string.Empty;
    }
}
```

we have kept the user model simple for time being for the same of implementing db later on we will add more columns/ attributes to it.

the error in AppDbContext.cs might remain and you will have to import the UserModel.cs file, you can do this by simply putting your cursor to the error(in vs code) and press alt+enter this will show you all the suggestions simply accept the suggestion to import it. below screenshot show it

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757466433738/97fa6d3d-94f8-4ed4-b37e-e20508b579d3.png align="center")

### iii. Configure Db Connection string

now we need to connect db to our backend server, for that we need to get the database connection string, here you have 3 options

1. set up postgres on your machine
    
2. set up postgres locally in docker
    
3. use a cloud provider like [neon.tech](https://neon.com/docs/connect/connect-from-any-app), the cloud option is fastest to setup but it is recommended to set it up locally.
    

how we will add the following to our appSettings.development.json(in production environment variables are saved separately), below we have just added the connection string.

```plaintext
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft.AspNetCore": "Warning"
    }
  }
  ,
"ConnectionStrings": {
    "PgConnectionString":"Host=localhost;Port=5432;Database=postgres;Username=postgres;Password=admin;Timeout=15;"}
}
```

### iv. register everything in program.cs file for things to work

add the following builder before builder.build()

```plaintext
builder.Services.AddDbContext<AppDbContext>(options =>
{
    options.UseNpgsql(builder.Configuration.GetConnectionString("PgConnectionString"));
});

```

### v. use ef core to actually make the tables in db.

now we simply need to run migrations and update database to add the tables( using our models and appDbContext.cs file)

go to the integrated terminal and type the following:

```plaintext
dotnet ef migrations add "user table"
```

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757468413048/b29a7e8e-f904-4727-8ea6-a5d7e4ad0c52.png align="center")

now we can move to actually creating the tables in db, run the following command:

```plaintext
dotnet ef database update
```

in the database we can actually see the table, with same attributes as our model

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757468626381/877d2f3a-51c0-478d-9941-564a7f8beef0.png align="center")

## Creating the CRUD endpoints for testing.

now for the purpose of testing we will create an end point and do CRUD(create, read, update, delete) on it.

simply create a Controllers folder and create an ApiController named, UserController

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757468970789/655b8c27-ca02-424d-ad00-56a1169ce8c1.png align="center")

add the following code in your Usercontroller:

```plaintext
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using dotnet_auth_with_db.Data;
using dotnet_auth_with_db.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.VisualBasic;

namespace dotnet_auth_with_db.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UserController(AppDbContext context) : ControllerBase
    {

        private readonly AppDbContext _context = context;

        [HttpGet]
        public IActionResult GetALlUsers()
        {

            var allUsers = _context.Users.ToList();
            if (allUsers.Count > 0)
            {
                return Ok(allUsers);
            }
            return NotFound();
        }
        [HttpPost]
        public async Task<IActionResult> CreateUser([FromBody] UserModel req)
        {
            var user = _context.Users.FirstOrDefault(u => u.Email == req.Email);

            if (user is not null)
            {
                return BadRequest("email already exists");
            }


            var newUser = new UserModel();
            newUser.Id = Guid.NewGuid();
            newUser.Email = req.Email;
            newUser.Password = req.Password;// hash this password in production

            _context.Users.Add(newUser);
            await _context.SaveChangesAsync();
            return Ok(newUser);

        }
    }
}
```

how using postman or httpie if we try creating a user, it does creates a new user and returns the user details:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757472269321/0b8a0a8d-c107-43d5-bd02-3e9a75798082.png align="center")

  
we can also check in the database that our user is created:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757472380241/f85f00bf-7c40-49a1-82bd-75a660a29554.png align="center")

now that we have created some users we want to fetch all users:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757473247747/f34a5401-3da6-433c-b43e-b1f60b4e8a6f.png align="center")

now we want to implement delete user and update user in the same userController.cs

```plaintext
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using dotnet_auth_with_db.Data;
using dotnet_auth_with_db.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.VisualBasic;

namespace dotnet_auth_with_db.Controllers
{
    [ApiController]
    [Route("api/[controller]")]
    public class UserController(AppDbContext context) : ControllerBase
    {

        private readonly AppDbContext _context = context;

        [HttpGet]
        public IActionResult GetALlUsers()
        {

            var allUsers = _context.Users.ToList();
            if (allUsers.Count > 0)
            {
                return Ok(allUsers);
            }
            return NotFound();
        }
        [HttpPost]
        public async Task<IActionResult> CreateUser([FromBody] UserModel req)
        {
            var user = _context.Users.FirstOrDefault(u => u.Email == req.Email);

            if (user is not null)
            {
                return BadRequest("email already exists");
            }


            var newUser = new UserModel();
            newUser.Id = Guid.NewGuid();
            newUser.Email = req.Email;
            newUser.Password = req.Password;// hash this password in production

            _context.Users.Add(newUser);
            await _context.SaveChangesAsync();
            return Ok(newUser);

        }
        [HttpPut]
        public async Task<IActionResult> UpdateAsync([FromBody] UserModel req)
        {
            var user = await _context.Users.FirstOrDefaultAsync(u => u.Id == req.Id);
            if (user is null)
            {
                return NotFound();
            }
            user.Email = req.Email;
            user.Password = req.Password;
            await _context.SaveChangesAsync();
            return Ok(user);
        }
        [HttpDelete]
        public async Task<IActionResult> DeleteUser([FromBody] Guid userId)
        {
            var user = _context.Users.FirstOrDefault(u => u.Id == userId);
            if (user is null)
            {
                return NotFound();
            }
            _context.Users.Remove(user);
            await _context.SaveChangesAsync();
            return NoContent();
        }
    }

}
```

how we can update and delete users as you can see from the below screenshots:

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757474538554/3b071ffb-516c-437a-ad1c-6a038535cddb.png align="center")

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1757474605766/61457d06-6f5c-4c3f-b18c-fc052faa3e7b.png align="center")

we have tested our endpoints, all the changes are reflected in the database.

the next step is to implement user authentication and authorization.