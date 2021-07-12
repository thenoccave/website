# Freeswitch Callcenter SMS Alerts
We recently deployed Freeswitch as our phone system. We are using mod_callcenter to manage our call queues. I wrote the below php script to send out an SMS alert if a call is sitting in a queue and there are no agents logged in. We are using Clickatell as our SMS gateway however you just need to modify the sendnotification function to a different provider. Just change the globals variables at the top to make it work for your setup
```
#!/usr/bin/php
 
<?php
 
        $GLOBALS['server'] = 'localhost';
        $GLOBALS['port'] = '8021';
        $GLOBALS['password'] = 'ClueCon';
        $GLOBALS['clickatellapi_id'] = '1234456';
        $GLOBALS['clickatelluser'] = 'username';
        $GLOBALS['clickatellpassword'] = 'password';
        $GLOBALS['contactnumbers'][] = '61123456789';
        $GLOBALS['contactnumbers'][] = '61987654321';
 
        $queuealerter = array();
 
        require_once('/usr/local/freeswitch/scripts/ESL.php');
 
        function sendnotification($queuename){
 
                $message = 'There is a call in the queue ' . $queuename . ' and no agents are logged in to take it';
 
                for($i=0; $i < count($GLOBALS['contactnumbers']); $i++){
 
                        $curl_handle = curl_init();
                        curl_setopt($curl_handle, CURLOPT_URL,'http://api.clickatell.com/http/sendmsg?api_id=' . $GLOBALS['clickatellapi_id'] . '&user=' . $GLOBALS['clickatelluser'] . '&password=' . $GLOBALS['clickatellpassword'] . '&to=' . $GLOBALS['contactnumbers'][$i] . '&text=' . urlencode($message));
                        curl_exec($curl_handle);
                        curl_close($curl_handle);
 
                }
 
 
        }
 
        $sock = new ESLconnection($GLOBALS['server'], $GLOBALS['port'], $GLOBALS['password']);
 
        $res = $sock->api('callcenter_config queue list');
        $queuelist = preg_split('/\r\n|\r|\n/', $res->getBody());
 
        //get a list of all the queues
        for($i=1; $i < count($queuelist); $i++){
 
                $queuedetails = explode('|', $queuelist[$i]);
                if($queuedetails[0] != '+OK' && $queuedetails[0] != ''){
 
 
                        $newQueue = array();
                        $newQueue['Name'] = $queuedetails[0];
                        $newQueue['AgentCount'] = 0;
                        $newQueue['CallCount'] = 0;
 
                        $queuealerter['Queues'][] = $newQueue;
 
                }
        }
 
        //for each queue check if there is a logged in agent
        for($i=0; $i < count($queuealerter['Queues']); $i++){
 
                $res = $sock->api('callcenter_config queue list agents ' . $queuealerter['Queues'][$i]['Name']);
                $agentdetails = preg_split('/\r\n|\r|\n/', $res->getBody());
 
                for($i2=1; $i2 < count($agentdetails); $i2++){
 
                        $agentdetail = explode('|', $agentdetails[$i2]);
                        if($agentdetail[0] != '+OK' && $agentdetail[0] != '' && $agentdetail[5] == 'Available'){
 
                                $queuealerter['Queues'][$i]['AgentCount'] += 1;
 
                        }
                }
 
                //get all the calls in the queue
 
                $res = $sock->api('callcenter_config queue list members ' . $queuealerter['Queues'][$i]['Name']);
                $calldetails = preg_split('/\r\n|\r|\n/', $res->getBody());
 
                for($i2=1; $i2 < count($calldetails); $i2++){
 
                        $calldetail = explode('|', $calldetails[$i2]);
                        if($calldetail[0] != '+OK' && $calldetail[0] != ''){
 
                                $queuealerter['Queues'][$i]['CallCount'] += 1;
 
                        }
                }
 
 
        }
 
        //for each queue check
        for($i=0; $i < count ($queuealerter['Queues']); $i++){
 
                if($queuealerter['Queues'][$i]['CallCount'] > 0 && $queuealerter['Queues'][$i]['AgentCount'] == 0){
                        sendnotification($queuealerter['Queues'][$i]['Name']);
                }
 
        }
 
?>
```
We run this as a cron job every minute
```
* * * * * /usr/local/freeswitch/scripts/queuealerter.php
```