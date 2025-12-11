


DagNode: 代表DAG中的一个节点，包含节点的唯一标识符（id）、数据（data）、依赖节点（dependencies）以及条件检查函数（conditionCheck）。节点可以依赖于其他节点，并且可以设置条件来决定节点是否可以被执行。
DagManager: 负责管理所有的DagNode，提供节点的添加、删除、查询、拓扑排序等功能。它还负责检测循环依赖，并确保DAG的正确性。
FlowManager: 使用DagManager来管理应用的导航流程。它根据用户偏好（如是否同意隐私政策、是否登录等）来决定当前应该显示的界面。
 

## 拓扑排序


## Kahn 算法


https://juejin.cn/post/7491963395284598821
https://github.com/jelychow/DAG


 -> Privacy Policy -> Terms of Service -> Register -> Profile -> Dashboard -> Settings -> Logout



Home -> Login -> Register

Home -> More

Home -> DeviceAdd

Settings -> Login -> Privacy Policy

Settings -> Logout

Settings -> DeviceManager -> DeviceAdd



