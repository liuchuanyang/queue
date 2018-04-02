# queue
c++ 缓存队列的实现
c++的队列缓存主要用于解决大数据量并发时的数据存储问题,可以将并发时的数据缓存到队列中,当数据量小时再匀速写入磁盘中

定义缓存结构体:
class DataInfo
{
public:
	DataInfo():pBuffer(NULL), iSize(0)
	{
	}
public:
   char *pBuffer; //缓存内容
   size_t iSize; //缓存大小
  
};

在头文件中实例化缓存队列:

queue<DataInfo> m_dq_buff; //初始化
	
创建管理队列的临界区

CRITICAL_SECTION m_lock;

InitializeCriticalSection(&m_lock); //初始化


缓存数据:

在数据回调函数或采集线程中进行数据缓存

void CallBack(int iType, char *pData, size_t len, void *pClass)
{
	CMyClass *pThis = (CMyClass*)pClass;
	DataInfo dataInfo; //实例化缓存结构体
	char *pbuf = new char[1024*1024*2]; //分配2MB的存储空间
	//缓存推送队列
	memcpy(pBuf, pdata, len); // 数据拷贝到缓存中
	
	dataInfo.pBuf = pBuf;
	dataInfo.iSize = pThis->_length;
	
	//使用临界区加锁
	EnterCriticalSection(&pThis->m_lock); //进入临界区
	pThis->m_dq_buf.push(dataInfo); //数据缓存推送到队列里
	
	LeaveCriticalSection(&pThis->m_lock); //退出临界区
}


数据处理:
 
 创建数据处理线程
 HANDLE m_hThread = CreateThread(NULL, 0, thread_work, this, 0, NULL);
 
 开始处理数据
 
 DWORD WINAPI thread_work(LPVOID lpParmeter)
 {
 	CMyClass *pThis = (CMyClass*)lpParmeter;
	DataInfo dataInfo; //实例化缓存结构体
	// 当缓存队列中的数据大于0时, 不断将数据取出进行处理
	while(pThis->m_dq_buf.size.size() > 0)
	{
		dataInfo = pThis->m_dq_buf.front();
		CheckData(dataInfo); //数据处理函数对数据进行处理或存储
		delete dataInfo.pBuf; //数据处理完成释放内存
		
		//使用临界区加锁
		EnterCriticalSection(&pThis->m_lock); //进入临界区
		pThis->m_dq_buf.pop();//将缓存从队列中删除
		LeaveCriticalSection(&pThis->m_lock);//退出临界区
	}
 }
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 
 


















