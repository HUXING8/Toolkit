# Apollo Map

- [简介](#简介)
- [地图格式](#地图格式)
  - [hd_map](#🗺️hd_map)
  - [routing_map](#🗺️routing_map)
  - [pnc_map](#🗺️pnc_map)
- [地图转换](#地图转换)
- [可视化](#地图格式)

## 简介
* Apollo中的地图有**hd_map**,**routing_map**,**pnc_map**等，所有子模块下的地图(如routing_map)皆由原始的hd_map生成。
* base_map中的字段为**Apollo Opendrive**格式。

## 地图格式

### 🗺️**hd_map**
Apollo中可以读取`.xml`和`.pb`格式的地图文件，读取地图的函数位于工程下的"modules/map/hdmap/hdmap_impl.h"，在其类`HDMapImpl`的中创建了protobuf格式的成员变量`map`，其字段如下所示：
```
// 位于modules/common_msgs/map_msgs
message Map {
  optional Header header = 1;        //地图基本信息，包含版本、时间等
  repeated Crosswalk crosswalk = 2;  //人行横道
  repeated Junction junction = 3;    //交叉路口
  repeated Lane lane = 4;           //车道
  repeated StopSign stop_sign = 5;  //停车标志
  repeated Signal signal = 6;       //信号灯
  repeated YieldSign yield = 7;     //让车标志
  repeated Overlap overlap = 8;     //重叠区域
  repeated ClearArea clear_area = 9;  //禁止停车区域
  repeated SpeedBump speed_bump = 10;  //减速带
  repeated Road road = 11;             //道路
  repeated ParkingSpace parking_space = 12; //停车区域
  repeated Sidewalk sidewalk = 13;  //路边的小路，或者行人走的路
}
```

<details>
  <summary>查看Lane和Road字段</summary>

  ```
  // A lane is part of a roadway, that is designated for use by a single line of vehicles.
  // Most public roads (include highways) have more than two lanes.
  message Lane {
    optional Id id = 1;         //编号

    // Central lane as reference trajectory, not necessary to be the geometry central.
    optional Curve central_curve = 2;     //中心曲线

    // Lane boundary curve.
    optional LaneBoundary left_boundary = 3;          //左边界
    optional LaneBoundary right_boundary = 4;         //右边界

    // in meters.
    optional double length = 5;                       //长度

    // Speed limit of the lane, in meters per second.
    optional double speed_limit = 6;           //速度限制

    repeated Id overlap_id = 7;                //重叠区域id

    // All lanes can be driving into (or from).
    repeated Id predecessor_id = 8;           //前任id
    repeated Id successor_id = 9;             //继任者id

    // Neighbor lanes on the same direction.
    repeated Id left_neighbor_forward_lane_id = 10;    //左边相邻前方车道id
    repeated Id right_neighbor_forward_lane_id = 11;   //右边相邻前方车道id

    enum LaneType {               //车道类型
      NONE = 1;                  //无
      CITY_DRIVING = 2;           //城市道路
      BIKING = 3;                 //自行车
      SIDEWALK = 4;               //人行道
      PARKING = 5;                //停车
    };
    optional LaneType type = 12;         //车道类型

    enum LaneTurn {
      NO_TURN = 1;        //直行
      LEFT_TURN = 2;      //左转弯
      RIGHT_TURN = 3;     //右转弯
      U_TURN = 4;         //掉头
    };
    optional LaneTurn turn = 13;          //转弯类型

    repeated Id left_neighbor_reverse_lane_id = 14;       //左边相邻反方向车道id
    repeated Id right_neighbor_reverse_lane_id = 15;      //右边相邻反方向车道id

    optional Id junction_id = 16;

    // Association between central point to closest boundary.
    repeated LaneSampleAssociation left_sample = 17;      //中心点与最近左边界之间的关联
    repeated LaneSampleAssociation right_sample = 18;     //中心点与最近右边界之间的关联

    enum LaneDirection {
      FORWARD = 1;     //前
      BACKWARD = 2;    //后，潮汐车道借用的情况？
      BIDIRECTION = 3;  //双向
    }
    optional LaneDirection direction = 19;   //车道方向

    // Association between central point to closest road boundary.
    repeated LaneSampleAssociation left_road_sample = 20;    //中心点与最近左路边界之间的关联
    repeated LaneSampleAssociation right_road_sample = 21;    //中心点与最近右路边界之间的关联
  }
  ```
  
  ```
  // road section defines a road cross-section, At least one section must be defined in order to
  // use a road, If multiple road sections are defined, they must be listed in order along the road
  message RoadSection {
    optional Id id = 1;
    // lanes contained in this section
    repeated Id lane_id = 2;
    // boundary of section
    optional RoadBoundary boundary = 3;
  }

  // The road is a collection of traffic elements, such as lanes, road boundary etc.
  // It provides general information about the road.
  message Road {
    optional Id id = 1;
    repeated RoadSection section = 2;

    // if lane road not in the junction, junction id is null.
    optional Id junction_id = 3;
  }
  ```
  </details>

Apollo通过`HDMapImpl`中不同的函数进行读取hd_map，根据参数格式进行选择：
```
// map_filename为本地地图文件路径
int HDMapImpl::LoadMapFromFile(string& map_filename)

//map_proto为一个protobuf格式的消息
int HDMapImpl::LoadMapFromProto(Map& map_proto)
```

`LaneInfo`类位于modules/map/hdmap/hdmap_common.h中，其中包括计算航向角、曲率、投影点、ST坐标等的功能函数。从map_lane.pb的`Lane`信息转化到`LaneInfo`在其构造函数中。
```
LaneInfo::LaneInfo(const Lane& lane) : lane_(lane) { Init(); }
```
Init()函数中初始化了`LaneInfo`中的segment_,accumulated_s_,unit_directions_,heading_变量，即车道离散中心线的线段、累积s长度，单位向量，航向角的数据，并在最后调用类内函数CreateKDTree()进行KDTree的构建，便于后续道路段segment的搜索。

### 🗺️**routing_map**
routing地图主要依靠一个TopoGraph来进行实现，将hd_map中的道路设置为Node，车道与车道之间的连接设置为Edge。<br>
<br>
topo_graph在"modules/routing/graph/topo_graph.h"中的`TopoGraph`类中进行初始化实现，其中含有`TopoNode`和`TopoEdge`的数组变量来存储拓扑图中的结点和边，使用类内函数LoadNodes(Graph& graph)和LoadEdges(Graph& graph)进行获取。<br>
<br>
其中的`Graph`数据为protobuf格式，字段来自"modules/routing/proto/topo_graph.proto"，具体内容如下：

```
message Node {
  optional string lane_id = 1;
  optional double length = 2;
  repeated CurveRange left_out = 3;
  repeated CurveRange right_out = 4;
  optional double cost = 5;
  optional apollo.hdmap.Curve central_curve = 6;
  optional bool is_virtual = 7 [default = true];
  optional string road_id = 8;
}

message Edge {
  enum DirectionType {
    FORWARD = 0;
    LEFT = 1;
    RIGHT = 2;
  }
  optional string from_lane_id = 1;
  optional string to_lane_id = 2;
  optional double cost = 3;
  optional DirectionType direction_type = 4;
}

message Graph {
  optional string hdmap_version = 1;
  optional string hdmap_district = 2;
  repeated Node node = 3;
  repeated Edge edge = 4;
}
```

graph的首次初始化位于"modules/routing/core/navigator.h"中的`Navigator`类中，该类为拓扑图的导航类，实现routing的路线搜索。<br>
<br>
在Navigator的构造函数Navigator(const string& topo_file_path)中，参数输入为topo_graph的本地文件路径，函数内调用GetProtoFromFile()来获取本地文件中的graph信息，并使用一个TopoGraph类的类内变量graph_，调用其LoadGraph()来初始化各结点和边信息。

### 🗺️**pnc_map**
`pnc_map`是Planning中的依据地图，在工程中创建了类`PncMap`，位于"modules/map/pnc_map/pnc_map.h"，主要用于后续规划时的参考线提供。


## 地图转换
将**OpenDrive**格式的地图转换成**Apollo Opendrive**格式 [imap](https://github.com/daohu527/imap)

## 可视化
* `.xml`格式地图文件可视化 [Apollo opendrive xml格式高精地图解析引擎](https://github.com/chenyongzhe/HdmapEngine) 
* `.txt`格式地图文件可视化 [Apollo_txt地图可视化](https://github.com/HUXING8/Apollo-Map-Read)

⬅️[返回](../ReadMe.md)