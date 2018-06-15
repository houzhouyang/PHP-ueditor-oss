//修改该方法即可



    private function upFile()
       {
        $file = $this->file = $_FILES[$this->fileField];
        if (!$file) {
            $this->stateInfo = $this->getStateInfo("ERROR_FILE_NOT_FOUND");
            return;
        }
        if ($this->file['error']) {
            $this->stateInfo = $this->getStateInfo($file['error']);
            return;
        } else if (!file_exists($file['tmp_name'])) {
            $this->stateInfo = $this->getStateInfo("ERROR_TMP_FILE_NOT_FOUND");
            return;
        } else if (!is_uploaded_file($file['tmp_name'])) {
            $this->stateInfo = $this->getStateInfo("ERROR_TMPFILE");
            return;
        }
        $this->oriName = $file['name'];
        $this->fileSize = $file['size'];
        $this->fileType = $this->getFileExt();
        $this->fullName = $this->getFullName();
        $this->filePath = $this->getFilePath();
        $this->fileName = $this->getFileName();
        $dirname = dirname($this->filePath);

        //检查文件大小是否超出限制
        if (!$this->checkSize()) {
            $this->stateInfo = $this->getStateInfo("ERROR_SIZE_EXCEED");
            return;
        }

        //检查是否不允许的文件格式
        if (!$this->checkType()) {
            $this->stateInfo = $this->getStateInfo("ERROR_TYPE_NOT_ALLOWED");
            return;
        }

        //创建目录失败
        if (!file_exists($dirname) && !mkdir($dirname, 0777, true)) {
            $this->stateInfo = $this->getStateInfo("ERROR_CREATE_DIR");
            return;
        } else if (!is_writeable($dirname)) {
            $this->stateInfo = $this->getStateInfo("ERROR_DIR_NOT_WRITEABLE");
            return;
        }

        //移动文件
       if (!(move_uploaded_file($file["tmp_name"], $this->filePath) && file_exists($this->filePath))) { //移动失败
            $this->stateInfo = $this->getStateInfo("ERROR_FILE_MOVE");
        } else { //移动成功
           include (__DIR__."/../../../../../../../vendor/oss/autoload.php"); //加载oss SDK  修改为你的文件路径
           $object='admin/'.substr(str_shuffle("qwertyuiopasdfghjklzxcvbnm0123456789"),0,5).time().$this->fileType; //oss对应的文件名/地址
           $imageUrl = '';

           $ossClient = new oss\OssClient('Your accesskeyId', 'Your accesskeySecret ', 'Your endpoint');

           try {
         
               $ossClient->uploadFile('Your bucket', $object, $this->filePath, array());
               $this->fullName='https://***.oss-cn-shenzhen.aliyuncs.com/' . $object;  你的oss服务器地址+文件名

               // 返回成功信息
               $this->stateInfo = $this->stateMap[0];

           } catch(oss\core\OssException $e) {

               // 返回错误消息
               $this->stateInfo = $this->getStateInfo("ERROR_FILE_MOVE");
           }
        }


    }
