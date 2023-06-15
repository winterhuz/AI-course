

#include <NewPing.h>
#include <Ultrasonic.h>
#include <SimpleDHT.h> 
#include <Wire.h>


//超音波設定
Ultrasonic ultrasonic1(4,17);
Ultrasonic ultrasonic2(27,34);
Ultrasonic ultrasonic3(18,5);
Ultrasonic ultrasonic4(16,15);
int distance1;
int distance2;
int distance3;
int distance4;
int frontobstacle;

// 馬達引脚
#define L298N_IN1   32
#define L298N_IN2   33
#define L298N_IN3   14 
#define L298N_IN4   26
#define motorspeed1  120         //超過100會有點恐怖
#define motorspeed2  120         //新電池雙120，延遲350ms差不多
#define L298N_ENA   25
#define L298N_ENB   23

// 馬達转向
int direct = 0; //0上;1左;2下;3右
const int forward = HIGH;
const int backward = LOW;
int sequence =0;
void goforward();
void turnright();
void turnleft();
void pausee();
void goabit();

//溫溼度
int pinDHT11 = 19;//GPIOPIN
SimpleDHT11 dht11;
unsigned long previousMillis = 0;
const unsigned long interval = 3000; // 3 seconds
void showdht11();

// astar參數

int currentX = 5;
int currentY = 5;
int targetX = 9;
int targetY = 5;
int openListCount = 0;
int closedListCount = 0;


//建立地圖
#define GRID_SIZE 10
int Map[GRID_SIZE][GRID_SIZE];
unsigned long runtime;
void initializeMap();
void detect();
void printMap();

//astar 函式宣告
struct Node {
  int x;
  int y;
  int f;
  int g;
  int h;
  Node* parent;
};

Node* openList[GRID_SIZE * GRID_SIZE];
Node* closedList[GRID_SIZE * GRID_SIZE];
Node* currentNode;
bool isInClosedList(int x, int y);
void addToClosedList(Node* node) ;
int isInOpenList(int x, int y) ;
void addToOpenList(Node* node);
void addToOpenList(Node* node);
int calculateHeuristic(int x, int y);
void tracePath(Node* targetNode) ;
void expandNode(Node* currentNode, int offsetX, int offsetY);
void removeFromOpenList(int index);
void findPath(){
  // 创建起始节点
  Node* startNode = new Node;
  startNode->x = currentX;
  startNode->y = currentY;
  startNode->g = 0;
  startNode->h = calculateHeuristic(currentX, currentY);
  startNode->f = startNode->g + startNode->h;
  startNode->parent = NULL;
  
  // 将起始节点加入到开放列表
  addToOpenList(startNode);
  
  while (openListCount > 0) {
    // 选择开放列表中f值最小的节点
    Node* currentNode = openList[0];
    int currentIndex = 0;
    for (int i = 1; i < openListCount; i++) {
      if (openList[i]->f < currentNode->f) {
        currentNode = openList[i];
        currentIndex = i;
      }
    }
    
    // 将该节点从开放列表中移除
    removeFromOpenList(currentIndex);
    
    // 将该节点加入到封闭列表
    addToClosedList(currentNode);
    
    // 检查当前节点是否为目标节点
    if (currentNode->x == targetX && currentNode->y == targetY) {
      // 找到了路径，停止寻找
      tracePath(currentNode);
      return;
    }
    
    // 扩展当前节点的相邻节点
    expandNode(currentNode, 0, -1); // 上
    expandNode(currentNode, 0, 1); // 下
    expandNode(currentNode, -1, 0); // 左
    expandNode(currentNode, 1, 0); // 右
  }
  
  // 开放列表为空，无法找到路径
  Serial.println("No path found!");
}
void botfollowPath(Node* targetNode);



//I2C Slave sender receive
#define I2C_SLAVE_ADDR 0x0a
#define datasize 12
void requestEvent();
void receiveEvent(int numBytes);
byte humidity_temperature[2];  //傳送陣列
byte data[datasize];           //接收陣列

//座標
int x,y;


//主程式開始

void setup()
{
    Wire.begin(I2C_SLAVE_ADDR);
    Wire.onReceive(receiveEvent);
    Wire.onRequest(requestEvent);
    Serial.begin(9600);
    initializeMap();
    pinMode(L298N_IN1, OUTPUT);
    pinMode(L298N_IN2, OUTPUT);
    pinMode(L298N_IN3, OUTPUT);
    pinMode(L298N_IN4, OUTPUT);
    digitalWrite(L298N_IN1, LOW);
    digitalWrite(L298N_IN2, LOW);
    digitalWrite(L298N_IN3, LOW);
    digitalWrite(L298N_IN4, LOW);
    runtime = millis();
}


void loop()
{
    //正常自走並繪製地圖

    //感測到目標物後執行astar
       if(1){
          findPath();
          botfollowPath(currentNode);
          }
      else{

    if ((frontobstacle == 1) && (sequence == 0))
    {
        turnright();
        pausee();
        goabit();
        pausee();
        turnright();
        sequence = ((sequence + 1) % 2);
    }
    else if(frontobstacle == 1 && sequence == 1)
    {
        turnleft();
        pausee();
        goabit();
        pausee();
        turnleft();
        sequence = ((sequence + 1) % 2);
    }
    goforward();
        }
    unsigned long currentMillis = millis();
    if (currentMillis - previousMillis >= interval)
    { 
   
        showdht11();
        previousMillis = currentMillis;
    }
// 定時打印地图
    if (millis() >= runtime + 3000)
    {
       printMap();
       runtime = millis();
    }
}




void expandNode(Node* currentNode, int offsetX, int offsetY) {
  int x = currentNode->x + offsetX;
  int y = currentNode->y + offsetY;
  
  // 检查坐标是否在地图范围内
  if (x < 0 || x >= GRID_SIZE || y < 0 || y >= GRID_SIZE) {
    return;
  }
  
  // 检查该坐标是否为障碍物或已在封闭列表中
  if (Map[x][y] == 3 || isInClosedList(x, y)) {
    return;
  }
  
  // 计算该节点的g、h和f值
  int g = currentNode->g + 1;
  int h = calculateHeuristic(x, y);
  int f = g + h;
  
  // 检查该节点是否已在开放列表中
  int existingIndex = isInOpenList(x, y);
  if (existingIndex != -1) {
    // 检查该节点是否有更优的路径
    if (g < openList[existingIndex]->g) {
      // 更新该节点的父节点和g、h、f值
      openList[existingIndex]->parent = currentNode;
      openList[existingIndex]->g = g;
      openList[existingIndex]->f = f;
    }
  } else {
    // 创建新节点，并设置父节点和g、h、f值
    Node* newNode = new Node;
    newNode->x = x;
    newNode->y = y;
    newNode->g = g;
    newNode->h = h;
    newNode->f = f;
    newNode->parent = currentNode;
    
    // 将新节点加入到开放列表
    addToOpenList(newNode);
  }
}


void addToOpenList(Node* node) {
  openList[openListCount] = node;
  openListCount++;
}

void removeFromOpenList(int index) {
  for (int i = index; i < openListCount - 1; i++) {
    openList[i] = openList[i + 1];
  }
  openListCount--;
}

void addToClosedList(Node* node) {
  closedList[closedListCount] = node;
  closedListCount++;
}

bool isInClosedList(int x, int y) {
  for (int i = 0; i < closedListCount; i++) {
    if (closedList[i]->x == x && closedList[i]->y == y) {
      return true;
    }
  }
  return false;
}

int isInOpenList(int x, int y) {
  for (int i = 0; i < openListCount; i++) {
    if (openList[i]->x == x && openList[i]->y == y) {
      return i;
    }
  }
  return -1;
}

int calculateHeuristic(int x, int y) {
  // 使用曼哈顿距离作为启发式函数
  return abs(targetX - x) + abs(targetY - y);
}


void tracePath(Node* targetNode) {
  Node* currentNode = targetNode;
  
  // 从目标节点回溯到起始节点，并打印路径
  while (currentNode != NULL) {
    Serial.print("[" + String(currentNode->x) + ", " + String(currentNode->y) + "] ");
    currentNode = currentNode->parent;
  }
  Serial.println();
}

void botfollowPath(Node* targetNode) {
  Node* currentNode = targetNode;
  int path_x[30];
  int path_y[30];
  int i;

  // 从目标节点回溯到起始节点，並複製路径至Path[]
  while (currentNode != NULL) {
    path_x[i]=currentNode->x;
    path_y[i]=currentNode->y;
    i++;
    currentNode = currentNode->parent;
  }
  for(;i>0;i--){

    if(path_x[i]-path_x[i-1]==1){ //往右移動
        switch(direct){
            case(0):{ turnright();goforward;break;}
            case(1):{goforward();break;}
            case(2):{ turnleft();goforward;break;}
            case(3):{ turnleft();turnleft();goforward();}break;}
    }
    else if(path_x[i]-path_x[i-1]==-1){  //往左移動
        switch(direct){
            case(0): {turnleft();goforward;break;}
            case(1): {turnleft();turnleft();goforward();break;}
            case(2): {turnright();goforward;break;}
            case(3): {goforward();break;}}
    } 
    else if(path_y[i]-path_y[i-1]==1){  //往上移動
        switch(direct){
            case(0): {goforward();break;}
            case(1): {turnleft();goforward;break;}
            case(2): {turnleft();turnleft();goforward();break;}
            case(3): {turnright();goforward;break;}}
    }
    else if(path_y[i]-path_y[i-1]==-1){  //往下移動
        switch(direct){
            case(0): {turnleft();turnleft();goforward();break;}
            case(1): {turnright();goforward;break;}
            case(2): {goforward();break;}
            case(3): {turnleft();goforward;break;}}
    }
  }
  Serial.println();
}




void initializeMap()
{
    for (int i = 0; i < GRID_SIZE; i++)
    {
        for (int j = 0; j < GRID_SIZE; j++)
        {
            Map[i][j] = 0; // 初始化地图，所有区域都未探测过
        }
    }
    for(int k =0; k<GRID_SIZE;k++)
    { 
      Map[GRID_SIZE-1][k]=3;
      Map[0][k]=3;
      Map[k][GRID_SIZE-1]=3;
      Map[k][0]=3;
    }
    printMap();
}



void detect()
{
    // 检测前方传感器状态
    // 0上;1左;2下;3右
    // 0未探索;1已走過;2無障礙物;3有障礙物
    distance1 = ultrasonic1.read();
    distance2 = ultrasonic2.read();
    distance3 = ultrasonic3.read(); //左
    distance4 = ultrasonic4.read(); //右
    Serial.print("Distance in CM: \n");
    Serial.print(distance1);
    Serial.print(",");
    Serial.println(distance2);
    Serial.print(",");
    Serial.println(distance3);
    Serial.print(",");
    Serial.println(distance4);

    // 檢測前方狀態
    if (distance1 < 20 || distance2 < 20)
    {
        frontobstacle = 1;    // 回報前方狀態
        switch (direct)
        {
            case 0:{
                Map[currentX][currentY + 1] = 3;break;} // 前方有障礙物
            case 1:{
                Map[currentX - 1][currentY] = 3;break;}
            case 2:{
                Map[currentX][currentY - 1] = 3;break;}
            case 3:{
                Map[currentX + 1][currentY] = 3;break;}
        }
  } else
    {
        frontobstacle = 0;    // 回報前方狀態
        switch (direct){
     case 0:{
      if(Map[currentX][currentY + 1] == 3){
        frontobstacle =1;break;}
      else{
        Map[currentX][currentY + 1] = 2;break;}} // 前方無障礙物
     case 1:{
      if(Map[currentX - 1][currentY] ==3){
        frontobstacle=1;break;}
      else{
        Map[currentX - 1][currentY] = 2;break;} }
     case 2: {
      if(Map[currentX][currentY - 1] ==3){
        frontobstacle=1;break;}
      else{
        Map[currentX][currentY - 1] = 2;break;}}
     case 3:{
      if(Map[currentX + 1][currentY] ==3){
        frontobstacle=1;break;}
      else{Map[currentX + 1][currentY] = 2;break;}
    }
    }
    }

    // 检测左方传感器状态
    if (distance3 < 20)
    {
        switch (direct)
        {
        case 0:{
            Map[currentX - 1][currentY] = 3;break;} // 方左有障礙物
        case 1:{
            Map[currentX][currentY + 1] = 3;break;}
        case 2:{
            Map[currentX + 1][currentY] = 3;break;}
        case 3:{
            Map[currentX][currentY - 1] = 3;break;}
        }   
    } else
    {
        switch(direct){
     case 0:{
      if(Map[currentX - 1][currentY]==3){
        frontobstacle=1;break;}
      else{
        Map[currentX - 1][currentY] = 2;break;}} // 方左無障礙物
     case 1:{
      if(Map[currentX][currentY + 1]==3){
        frontobstacle=1;break;}
      else
     Map[currentX][currentY + 1] = 2; break;}
     case 2:{ 
      if(Map[currentX + 1][currentY]==3){
        frontobstacle=1;break;}
      else{
        Map[currentX + 1][currentY] = 2;break;}}
     case 3:{
      if(Map[currentX][currentY - 1]==3){
        frontobstacle=1;break;}
      else{
        Map[currentX][currentY - 1] = 2;break;}}
        }
    }

    // 检测右方传感器状态
    if (distance4 < 20)
    {
    switch (direct)
    {
        case 0:{
            Map[currentX + 1][currentY] = 3;break;}// 右方有障礙物
        case 1:{
            Map[currentX][currentY - 1] = 3;break;}
        case 2:{
            Map[currentX - 1][currentY] = 3;break;}
        case 3:{
            Map[currentX][currentY + 1] = 3;break;}
    }
    } else
    {
        switch (direct)
        {
            case 0:{
             if(Map[currentX + 1][currentY]==3){
               frontobstacle=1;break;}
             else{
               Map[currentX + 1][currentY] = 2;break;}}// 右方有障礙物
            case 1:{
              if(Map[currentX][currentY - 1]==3){
               frontobstacle=1;break;}
             else{
               Map[currentX][currentY - 1] = 2;break;}}
            case 2:{
              if(Map[currentX - 1][currentY]==3){
               frontobstacle=1;break;}
             else{
               Map[currentX - 1][currentY] = 2;break;}}
            case 3:{
              if(Map[currentX][currentY + 1]==3){
               frontobstacle=1;break;}
             else{
               Map[currentX][currentY + 1] = 2;break;}}
        }
    }
}


void printMap(){
    for (int i = 0; i < GRID_SIZE; i++)           
        {       
            for (int j = 0; j < GRID_SIZE; j++)            
            {            
                printf("%d",Map[i][j]);                
                printf("_  _");                
            }            
            printf("\n");            
        }        
    } 
void goforward(){    //前進函式
    analogWrite(L298N_ENA, motorspeed1);        
    analogWrite(L298N_ENB, motorspeed2);        
    digitalWrite(L298N_IN1, LOW);        
    digitalWrite(L298N_IN2, HIGH);        
    digitalWrite(L298N_IN3, HIGH);       
    digitalWrite(L298N_IN4, LOW);        
    delay(200);
    detect();
    switch (direct)
    {
        case 0:{
            currentY =currentY+ 1;break;}
        case 1:{
            currentX = currentX-1;break;}
        case 2:{
            currentY = currentY -1;break;}
        case 3:{
            currentX = currentX +1;break;}
    }
    printf("current(%d,%d) \n",currentX,currentY);
}
void turnright()       //向右轉
        {                    
            analogWrite(L298N_ENA, motorspeed1);            
            analogWrite(L298N_ENB, motorspeed2);           
            digitalWrite(L298N_IN1, HIGH);
                    digitalWrite(L298N_IN2, LOW);
                    digitalWrite(L298N_IN3, HIGH);
                    digitalWrite(L298N_IN4, LOW);
                    printf("turnR \n");
                    direct = (direct + 3) % 4;
                    delay(350);

                }
void turnleft()      //向左轉
                {
                    analogWrite(L298N_ENA, motorspeed1);
                    analogWrite(L298N_ENB, motorspeed2);
                    digitalWrite(L298N_IN1, LOW);
                    digitalWrite(L298N_IN2, HIGH);
                    digitalWrite(L298N_IN3, LOW);
                    digitalWrite(L298N_IN4, HIGH);
                    printf("turnL \n");
                    direct = (direct + 1) % 4;
                    delay(350);

                }
void pausee()         //停止
                {
                    analogWrite(L298N_ENA, motorspeed1);
                    analogWrite(L298N_ENB, motorspeed2);
                    digitalWrite(L298N_IN1, LOW);
                    digitalWrite(L298N_IN2, LOW);
                    digitalWrite(L298N_IN3, LOW);
                    digitalWrite(L298N_IN4, LOW);
                    delay(1000);

                }

void goabit()
        {
            goforward();
        }
        
void showdht11(){
    byte temperature = 0;
    byte humidity = 0;
    int err = SimpleDHTErrSuccess;
    // start working...
    Serial.println("=================================");
    if ((err = dht11.read(pinDHT11, &temperature, &humidity, NULL)) != SimpleDHTErrSuccess)
    {
        Serial.print("Read DHT11 failed, err="); Serial.println(err); delay(1000);
        return;
    }
    humidity_temperature[0]=humidity;
    humidity_temperature[1]=temperature;
    Serial.print("Humidity = ");
    Serial.print((int)humidity);
    Serial.print("% , ");
    Serial.print("Temperature = ");
    Serial.print((int)temperature);
    Serial.println("C ");
    //不可快過每2秒顯示一次
}

void Coordinate_position()                      //得到座標
{
  Serial.print("接收到的資料：");                  //I2C接收資料
  int i;
    for(i=0;i<datasize-1;i++){
      Serial.print(data[i]);
      Serial.print("，");
    }
    Serial.println(data[datasize]);
    if(data[9]==1&&data[10]==1){                   //接收資料轉座標
      x=0-data[1]-data[2]*256;
      y=0-data[5]-data[6]*256;
    }
    if(data[9]==1&&data[10]==0){
      x=0-data[1]-data[2]*256;
      y=data[5]+data[6]*256;
    }
    if(data[9]==0&&data[10]==1){
      x=data[1]+data[2]*256;
      y=0-data[5]-data[6]*256;
    } 
    if(data[9]==0&&data[10]==0){
      x=data[1]+data[2]*256;
      y=data[5]+data[6]*256;
    }  
    Serial.printf("座標:%d,%d",x,y);  
    Serial.println("");
    Serial.println("============================");
}




void requestEvent() {
  Wire.write(humidity_temperature, sizeof(humidity_temperature));//I2C傳輸函式使用2byte傳輸
}
void receiveEvent(int numBytes){  //I2C接收函式
  int i;
  while (Wire.available()) {   // 在這裡處理接收到的資料
    for(i=0;i<datasize;i++)
      data[i] = Wire.read(); // 讀取datasize個字節
  }
}
