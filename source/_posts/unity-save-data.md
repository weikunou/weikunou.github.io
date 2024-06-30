---
title: Unity3D 存档读档
date: 2024-04-21 09:18:23
tags: Unity3D
---

Unity3D json 存档读档功能。

<!--more-->

# 序列化

游戏存档通常是把存在于内存中的对象，通过序列化，把对象中的数据存储到文件中，实现数据的持久化。

游戏读档则是把文件中的数据，通过反序列化，重新生成内存中的对象。

存档的格式，一般使用 json 存储，也可以使用二进制、XML等。

# 存档

首先，需要安装一个 json 库，在 Package Manager 窗口中，搜索 Newtonsoft Json 进行安装。

也可以使用其他 json 库，Newtonsoft Json 的优点是可以很好地序列化字典。

接着，创建一个 GameData 类，标记为可序列化。

```c#
using System;

[Serializable]
public class GameData
{
    public int playerID;

    public GameData()
    {
        playerID = 100001;
    }
}
```

然后创建一个 DataManager 类，用来管理存档。

这里的 SavePath 添加了平台宏定义，如果是在 unity editor 中运行，就直接存储到 Assets 文件夹中，其他平台则是存储到可读写的数据目录中。

通过调用 `JsonConvert.SerializeObject` 方法，把 gameData 序列化成 json 字符串，再使用 `StreamWriter` 写入本地文件。

```c#
using System.IO;
using UnityEngine;
using Newtonsoft.Json;

public class DataManager : MonoBehaviour
{
    public GameData gameData;

    string SavePath
    {
        get
        {
#if UNITY_EDITOR
            return Application.dataPath + "/SaveData.json";
#else
            return Application.persistentDataPath + "/SaveData.json";
#endif            
        }
    }

    void Start()
    {
        Save();
    }

    public void Save()
    {
        string json = JsonConvert.SerializeObject(gameData);
        StreamWriter sw = new StreamWriter(SavePath);
        sw.Write(json);
        sw.Close();
    }
}
```

运行游戏之后，停止运行，刷新工程资源目录，可以看到 Assets 文件夹下多出了一个 SaveData.json 文件。

json 的内容如下：

```json
{"playerID":100001}
```



# 读档

读档的时候，要先判断本地是否有存档文件，再进行文件的读取和反序列化，还原 gameData 数据对象。

```c#
public class DataManager : MonoBehaviour
{
    // ...
    
    public void Load()
    {
        if (File.Exists(SavePath))
        {
            StreamReader sr = new StreamReader(SavePath);
            string json = sr.ReadToEnd();
            sr.Close();
            gameData = JsonConvert.DeserializeObject<GameData>(json);
        }
    }
}
```

在游戏开始时，先读档，再修改数据对象，再次存档。

```c#
public class DataManager : MonoBehaviour
{
    // ...
    
    void Start()
    {
        Load();
        gameData.playerID = 100002;
        Save();
    }
    
    // ...
}
```

可以看到，json 的内容改变了。

```json
{"playerID":100002}
```

# 数据对象扩展

目前 GameData 中只有一个 playerID，实际上可以再定义更多的数据类，统一放到 GameData 中作为成员字段。

例如，定义一个 DataPlayer 类，保存玩家的数据。

```c#
public class DataPlayer
{
    public int playerID;
    public string playerName;
    public int health;

    public DataPlayer()
    {
        playerID = 100001;
        playerName = "Player";
        health = 100;
    }
}
```

然后把 DataPlayer 放到 GameData 中，在构造函数中要记得实例化对象。

```c#
using System;

[Serializable]
public class GameData
{
    public DataPlayer dataPlayer;

    public GameData()
    {
        dataPlayer = new DataPlayer();
    }
}
```

游戏开始时，修改一下玩家名字，再存档。

```c#
public class DataManager : MonoBehaviour
{
    // ...
    
    void Start()
    {
        Load();
        gameData.dataPlayer.playerName = "起个名字很难";
        Save();
    }
    
    // ...
}
```

此时，json 的内容如下：

```json
{
    "dataPlayer": {
        "playerID": 100001,
        "playerName": "起个名字很难",
        "health": 100
    }
}
```

