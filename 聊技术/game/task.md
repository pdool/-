![](..\..\img\20200926\1.png)

前言：一直在写Java 方面的，今天写一写游戏设计方面的 一任务系统。任务系统是每个任务的标配，不管是普通的小游戏，还是大型的端游，手游，没了任务系统的游戏是不完整的。任务系统是游戏的驱动，是玩家的目标，有了目标才会有追求，正常完成任务都会获得奖励，有糖吃，开心。

   **任务其实本身就是一种“激励制度”**，接受任务-进行挑战-完成奖励，这个过程中的每一个环节，都是对玩家游戏的激励，所以任务在其中就起着这样的这样的作用。对于玩家自身来说，从**心理学上**，完成任务是对自己挑战的肯定，会大大增加玩家游戏的满足感，即使没有顺利完成某一项任务，玩家感受到到了任务的难度，也会激起自己的挑战欲望和探索心理，进一步提升了游戏性。

###  1、任务的分类.

任务的需求就不多说了，先说下之前的SLG 游戏的任务类型。

根据任务类型分为 主线任务，支线任务，公会任务，势力任务，每日任务，还有各种不同的功能根据需求的任务类型

根据进度类型分为 累计型进度，递减型进度，达到型进度，激活型任务进度，自定义类型进度等等。

任务是游戏里比较复杂的系统，所以设计的时候应该尽可能的方便扩展，在增加任务的时候能简单的配置出来。

总结下设计原则：尽可能的解决问题，如果特殊需求可以有接口简单可以实现。

让我们操练起来吧！

### 2、基础数据的定义

配置是基础数据，是策划配置的，因此最简单的配置就是

任务id 一 任务达成条件一奖励

任务id 是测试自己创建唯一的id，任务达成条件是在游戏内任务的判断条件，当达到条件的时候发给玩家奖励。

### 3、系统的设计

根据任务的需求，我们对任务进行拆分。

![image-20200926210341620](../../\img\20200926\2.png)

根据进度类型进行拆分：

![image-20200926211015062](../../\img\20200926\3.png)

### 4、show me the fuck code ！

​	对任务拆分之后，现在得想办法实现我们的任务系统了。

首先先解决任务进度定义的问题：

#### 1.任务进度定义：

```
package org.pdool.task;

import com.google.common.base.Joiner;

public enum TaskProgress {
    /**
     * 格式：x_n
     * 专指完成某一任务几次（完成一次扣一次）
     * eq： 送花{n}次
     */
    PROGRESS_TYPE1 {
        @Override
        public String checkProgress(int roleId, String config, String current, Object paramArr) {
            String configId = config.split("_")[0];
            Object[] ext = (Object[]) paramArr;
            String param = ext[0].toString();
            long var = Long.parseLong(param);
            int needNum = Integer.parseInt(current.split("_")[1]);
            if (needNum > var) {
                return "";
            }
            return Joiner.on("_").join(configId, needNum - var);
        }
    },
    ;

    /**
     * 检测任务进度
     * @param roleId 玩家id
     * @param config 配置的任务条件
     * @param current 当前的任务进度
     * @param param 传进的参数
     * @return 当前的任务进度
     */
    public abstract String checkProgress(int roleId, String config, String current,  Object param);
}
```

#### 2.定义任务类型枚举

```

/**
 * @author 香菜
 */
public enum TaskTypeEnum {
    //  1:送花 eg:1_100
    SEND_FLOWER_1(TaskProgressEnum.PROGRESS_TYPE1, "1"),
    ;

    // 任务进度处理类型
    private TaskProgressEnum progressType;
    // 任务事件激活类型
    private String taskType;

    TaskTypeEnum(TaskProgressEnum taskType, String actType) {
        this.progressType = taskType;
        this.taskType = actType;
    }

    public TaskProgressEnum getProgressType() {
        return progressType;
    }

    public String getTaskType() {
        return taskType;
    }
}

```

策划可以配置的任务就是 1_100  表示 送花100 次则完成任务。

#### 3.定义任务模块枚举

举个例子，我们的创角七日任务，在服务器启动的时候加载就加载基础数据，并且注册到每日任务的处理器上，这样在有任务过来的，就不会分配到其他的任务上，因此我们实现一个处理器的抽象类，一个任务类型枚举。

```
/**
 * @author 香菜
 */
public enum TaskHandlerEnum {
    //  主线任务
    MAIN_TASK(1),
    //  日常任务
    DAILY_TASK(2),
    ;
    private int type;
    public int getType() {
        return type;
    }
    private TaskHandlerEnum(int type) {
        this.type = (byte) type;
    }
}
```

####  4.定义任务处理器的抽象类

至少得三个方法，一个构造函数，一个handle方法，一个注册方法。

```
/**
 * @author 香菜
 */
public abstract class AbsTaskHandler {
    public TaskHandlerEnum handlerEnum;
    private Map<TaskTypeEnum, Set<Integer>> canHandMap = Maps.newHashMap();
    public AbsTaskHandler(TaskHandlerEnum handlerEnum) {
        this.handlerEnum = handlerEnum;
    }
    /**
     * 注册任务到处理器
     * @param taskType
     * @param taskId
     */
    protected void registerTask(TaskTypeEnum taskType, int taskId) {
        Set<Integer> taskIdSet = canHandMap.computeIfAbsent(taskType, k -> Sets.newHashSet());
        taskIdSet.add(taskId);
    }

    /**
     * 每种任务类型单独实现
     * @param roleId
     * @param paramMap
     */
    protected abstract void handle(int roleId, Map<TaskTypeEnum, Object> paramMap);
}

```

#### 5.实现主线任务处理类

```
import java.util.Map;

public class MainTaskHandler extends AbsTaskHandler {
    public MainTaskHandler(TaskHandlerEnum handlerEnum) {
        super(handlerEnum);
    }

    @Override
    protected void handle(int roleId, Map<TaskTypeEnum, Object> paramMap) {
        //  do what you want
    }
}
```

#### 6.在服务器启动的时候将所有的处理器加入到map

Map<TaskHandlerEnum ,AbsTaskHandler>  handlerMap

#### 7.服务器启动后加载基础数据，将对应的任务注册到对应的处理器

registTaskToHanlder(TaskHandlerEnum,TaskType,TaskId)

#### 8.通知任务系统做检测

 sendMsgToTask（TaskType,param）

遍历所有的处理器，能处理这个这个任务，则处理，不能处理则过

### 5、总结

  由于任务系统的复杂性，设计上必然有一些挑战，但是这种系统如果理解了，也都是套路的问题，做的久了自然而然就会了，并且可以根据需求自由的调整，理解最重要。

  基本的介绍了任务系统的设计实现，后面一些没有做具体的展示，大家可以自己发挥，如果觉得有困难，并且又感兴趣的小伙伴可以联系我。或者你有其他的更好的设计我们可以一起交流。坚持写不容易，希望能获得大家的支持，点赞，转发 三连，谢谢。

