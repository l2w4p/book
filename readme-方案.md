flowchart TD
    %% 外部交互
    subgraph External[外部服务]
        direction TB
        S3[云存储（AWS S3 / 腾讯云 COS）]
        DB[(数据库：SQLite → PostgreSQL)]
    end

    %% 用户与客户端
    subgraph Client[客户端（PyQt5 桌面应用）]
        direction TB
        Login[登录界面] -->|验证身份| Auth[权限与会话管理]
        Auth --> MainUI[主操作界面]
        MainUI -->|手动输入| ManualInput[手动输入表单]
        MainUI -->|拍照按钮| CameraModule[拍照模块]
        CameraModule -->|调用摄像头| OpenCV[OpenCV 采图]
        OpenCV -->|预处理| ImgProc[图像预处理（去噪/增亮/透视校正）]
        ImgProc -->|送入 OCR| OCREngine[OCR 引擎（PaddleOCR / Tesseract）]
        OCREngine -->|识别结果+置信度| OCRResult[OCR 结果]
        OCRResult -->|置信度≥0.8| AutoExtract[自动字段提取（金额/日期）]
        OCRResult -->|置信度<0.8| ManualConfirm[手动确认框]
        ManualConfirm -->|用户修正| ManualInput
        AutoExtract -->|结构化数据| DataEngine[数据引擎]
        ManualInput -->|结构化数据| DataEngine
        DataEngine -->|更新今日记录| TodayRecord[今日记录列表]
        TodayRecord -->|实时计算| BalanceCalc[余额计算（Decimal）]
        BalanceCalc -->|显示| MainUI
        MainUI -->|借款管理| LoanMgr[客户借款管理]
        LoanMgr -->|待审核/已批准/已回收| LoanData[借款数据表]
        MainUI -->|结算触发器| SettleTimer[定时结算（10:00/16:00）]
        SettleTimer -->|生成结算单| SettleEngine[结算引擎]
        SettleEngine -->|写入| TodayRecord
        SettleEngine -->|更新| LoanData
        MainUI -->|日志记录| Logger[日志模块 (loguru)]
        Logger -->|写文件| LogFile[本地日志文件]
        MainUI -->|权限校验| Auth
        %% 云同步与离线缓存
        MainUI -->|后台任务| SyncTask[同步任务队列]
        SyncTask -->|加密照片+数据| Encryptor[本地 AES-256 加密]
        Encryptor -->|上传| S3
        S3 -->|生命周期规则| Glacier[低频归档（Glacier）]
        SyncTask -->|离线缓存| LocalQueue[本地操作队列]
        LocalQueue -->|联网后重放| SyncTask
        %% 数据库交互
        DataEngine -->|读写| DB
        LoanMgr -->|读写| DB
        SettleEngine -->|读写| DB
        Logger -->|可选异步写入| DB
    end

    %% 样式调整（可选）
    classDef external fill:#f9f9f9,stroke:#333,stroke-width:1px;
    classDef client fill:#e3f2fd,stroke:#1565c0,stroke-width:1.5px;
    class S3,DB external;
    class Login,Auth,MainUI,ManualInput,CameraModule,OpenCV,ImgProc,OCREngine,OCRResult,AutoExtract,ManualConfirm,DataEngine,TodayRecord,BalanceCalc,LoanMgr,LoanData,SettleTimer,SettleEngine,Logger,LogFile,SyncTask,Encryptor,LocalQueue,Glacier client;
