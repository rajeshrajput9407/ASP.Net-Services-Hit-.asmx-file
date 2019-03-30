# ASP.Net-Services-Hit-.asmx-file

```
 [WebMethod]
    public int CheckLicenseInfo(string LicenseKey, string ProductKey)
    {
        String status = string.Empty;
        int code = 0;
        int licenseInstalled = 0;
        int MaxLimit = 0;
        string licenseMachineIPs = "";

        try
        {
            IntelliLockDB.ILDataBase db = DBHelper.GetDataBase();
            IntelliLockDB.LicenseTrack data = db.LicenseTracker.GetLicenseTrackByLicenseTrackID(LicenseKey);
            if (data == null)
            {
                return -5;
            }
            NewDataSet result = GetAllLicenseTrackerRecords();
            for (int i = 0; i < result.LicenseTrack.Count; i++)
            {
                if (result.LicenseTrack[i].LicenseTrackID == LicenseKey)
                {
                    licenseInstalled = result.LicenseTrack[i].LicenseInstalled;
                    licenseMachineIPs = result.LicenseTrack[i].LicenseInstalled_SystemsIPS;
                    MaxLimit = Convert.ToInt32(result.LicenseTrack[i].FloatingLicense_MaxUser);
                }
            }
            if (data != null && data.ProductID == ProductKey)
            {

                if (licenseInstalled > MaxLimit)
                {
                    return -1;
                }
                if (data.BlockLicense == true)
                {
                    return -2;
                }
                if (data.ExpirationDate < System.DateTime.Now)
                {
                    return -3;
                }
                return 1;
            }
            else
            {
                return -5;
            }
        }
        catch (Exception ex)
        {
            using (StreamWriter writetext = new StreamWriter("write.txt"))
            {
                writetext.WriteLine(ex.Message+ex.StackTrace+ex.InnerException);
            }
            return -20;
        }
    }
    
    
    
    /////////////////  Service Hit

 WebRequest request = WebRequest.Create("http://192.168.1.113:8011/ValidationService.asmx/CheckLicenseInfo");
           request.Method = "POST";
            string postData = "LicenseKey="+ license+ "&ProductKey=" + ProductKey;
            byte[] dataBuffer = System.Text.Encoding.UTF8.GetBytes(postData);
            request.Proxy = new System.Net.WebProxy("http://192.168.1.113:8011");
            request.ContentType = "application/x-www-form-urlencoded";
            request.ContentLength = dataBuffer.Length;
            Stream postStream = request.GetRequestStream();
             postStream.Write(dataBuffer, 0, dataBuffer.Length);
            var response = (HttpWebResponse)request.GetResponse();  
            var responseString = new StreamReader(response.GetResponseStream()).ReadToEnd();
            XmlDocument doc = new XmlDocument();
            doc.LoadXml(responseString);
            string jsonText = JsonConvert.SerializeXmlNode(doc);
            XmlDocument doc3 = JsonConvert.DeserializeXmlNode(jsonText);
            var result = doc3.InnerText;
            MessageBox.Show(Messages.SoftwareMessages(Convert.ToInt32(result)), "Information", MessageBoxButtons.OK,    MessageBoxIcon.Information);
            postStream.Close();
            
            ```
    
