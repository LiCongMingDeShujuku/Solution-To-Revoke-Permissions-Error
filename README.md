![CLEVER DATA GIT REPO](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/0-clever-data-github.png "李聪明的数据库")

# 撤消权限错误的解决方案
#### Solution To SQL's Revoke Permissions Error
**发布-日期: 2015年02月26日 (评论)**

## Contents

- [Build Info](#Build-Info)
- [Author](#Author)
- [License](#License) 

### Error: Server principal ‘INN\Chris’ has granted one or more permission(s). Revoke the permission(s) before dropping the server principal.

错误：服务器主体“INN\Chris”已授予一个或多个权限。在删除服务器主体之前撤消权限。

You’ve run across this error before many times. Yes; it’s annoying, and then even more so when you have to run through the Securable Permissions under the Login Properties to resolve the problem. Why go through the GUI when you can simply run some quick SQL logic.

你经常多次遇到此错误。是挺烦人的，当你必须在登录属性下运行安全权限来解决问题时更是如此。当你可以简单地运行一些快速SQL逻辑时，为什么还要通过GUI呢。

![#](images/Solution-To-Revoke-Permissions-Error-01.png?raw=true "#")
 
What it looks like when you query it directly.

这是你直接查询时的样子

```SQL
use master;
set nocount on
 
select
       'account name'             = servprin.name
,      'type'                     = servprin.type_desc
,      'permission'               = servperm.permission_name
,      'state'                    = servperm.state_desc
from
       sys.server_principals servprin
       join sys.server_permissions servperm
       on  servprin.principal_id = servperm.grantee_principal_id
where
       servprin.type in ('s', 'u', 'g')
       and servprin.name like '%chris%'
order by
       'account name' asc
```

![#](images/Solution-To-Revoke-Permissions-Error-02.png?raw=true "#")
 
Once you can see the explicit permissions that were granted, all you need to do is run the following statement to remove (aka REVOKE) the permissions from the account so you can drop the login.

一旦你可以看到已授予的显式权限，你只需运行以下语句即可从帐户中删除（也称为REVOKE）权限，以便你可以删除登录。

Lets drop those CONNECT permissions from the account.

让我们一起来从帐户中删除那些CONNECT权限。

`revoke connect sql to [INN\Chris] as [sa];`

Next; we’ll need to determine what kind of GRANTS that Chris did under his account, then simply remove those granted permissions from those objects. You might want to notate what objects had the GRANT permissions applied to them and simply apply the GRANTS back to them using a more universal account.

下一步，我们需要确定Chris在他的帐户下做了什么样的授权，然后只需从这些对象中删除那些授予的权限。你如果想记录哪些对象已应用GRANT权限，只需使用更通用的帐户将GRANTS应用于它们。

```SQL
declare @grants     varchar(50)
set     @grants     = ( select principal_id from sys.server_principals where name = 'mydomain\myusername' )
select * from sys.server_permissions where grantor_principal_id = @grants

```
![#](images/Solution-To-Revoke-Permissions-Error-03.png?raw=true "#")
 
So now we know that Chris granted Connect Permissions to an Endpoint. This is where you need to be careful because even though we can simply revoke the permission; you kinda don’t want to break the connectivity to the Endpoint. The best thing to do is have another account take ownership of the Endpoint, and by doing so all corresponding permissions will simply be transferred to the new account.

所以现在我们知道Chris将Connect Permissions授予了端点。这是你需要小心的地方，因为即使我们可以简单地撤销许可，你也不会想破坏与端点的连接。最好的办法是让另一个帐户获得Endpoint的所有权，通过这样做，所有相应的权限将被简单地转移到新帐户。

First; lets see which endpoint that we are dealing with. So far all we have to go on is the ‘Major_ID’, but not the Endpoint name which we’ll need in order to run the ‘take ownership’ grant statement.

第一，让我们看看我们正在处理哪个端点。到目前为止，我们所要做的只是'Major_ID'，而不是我们为了运行'take ownership'grant语句而需要的Endpoint名称。

![#](images/Solution-To-Revoke-Permissions-Error-04.png?raw=true "#")
 
```SQL
	use master;
grant take ownership on endpoint::hadr_endpoint to sa
    with grant option;
go
```
Notice the ‘sa’. Unfortunately; you can’t do this. You may feel compelled to use a basic pre-existing account to get through this step, but before you think about using ‘sa’, consider this… It will error out. That’s just the way it’s designed.

注意。但是你不能这样做。你可能觉得有必要使用基本的预先存在的帐户来完成此步骤，但在你考虑使用“sa”之前，请考虑这一点，它会出错。这就是它的设计方式。

![#](images/Solution-To-Revoke-Permissions-Error-05.png?raw=true "#")
 
All you need do in this case is create another SQL Login. In this case; we’ll be using an SQL Login called EndpointOwner. Nothing special here, just a new account with sa rights. You can add something more complex with more granular rights later. The main goal here is to remove the accounts that need to be removed, and making sure you are giving ownership of the existing endpoint to another account so you don’t cause any problems by revoking the old one.

在这种情况下，你只需要创建另一个SQL登录。在这种情况下，我们将使用名为EndpointOwner的SQL登录。这里没什么特别的，只是一个拥有sa权限的新帐户。稍后你可以添加更复杂的内容以及更细分的权限。这里的主要目标是删除需要删除的帐户，并确保将现有端点的所有权授予另一个帐户，这样你就不会因撤销旧帐户而导致任何问题。

[![WorksEveryTime](https://forthebadge.com/images/badges/60-percent-of-the-time-works-every-time.svg)](https://shitday.de/)

## Build-Info

| Build Quality | Build History |
|--|--|
|<table><tr><td>[![Build-Status](https://ci.appveyor.com/api/projects/status/pjxh5g91jpbh7t84?svg?style=flat-square)](#)</td></tr><tr><td>[![Coverage](https://coveralls.io/repos/github/tygerbytes/ResourceFitness/badge.svg?style=flat-square)](#)</td></tr><tr><td>[![Nuget](https://img.shields.io/nuget/v/TW.Resfit.Core.svg?style=flat-square)](#)</td></tr></table>|<table><tr><td>[![Build history](https://buildstats.info/appveyor/chart/tygerbytes/resourcefitness)](#)</td></tr></table>|

## Author

- **李聪明的数据库 Lee's Clever Data**
- **Mike的数据库宝典 Mikes Database Collection**
- **李聪明的数据库** "Lee Songming"

[![Gist](https://img.shields.io/badge/Gist-李聪明的数据库-<COLOR>.svg)](https://gist.github.com/congmingshuju)
[![Twitter](https://img.shields.io/badge/Twitter-mike的数据库宝典-<COLOR>.svg)](https://twitter.com/mikesdatawork?lang=en)
[![Wordpress](https://img.shields.io/badge/Wordpress-mike的数据库宝典-<COLOR>.svg)](https://mikesdatawork.wordpress.com/)

---
## License
[![LicenseCCSA](https://img.shields.io/badge/License-CreativeCommonsSA-<COLOR>.svg)](https://creativecommons.org/share-your-work/licensing-types-examples/)

![Lee Songming](https://raw.githubusercontent.com/LiCongMingDeShujuku/git-resources/master/1-clever-data-github.png "李聪明的数据库")


