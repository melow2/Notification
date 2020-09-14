# Notification
안드로이드 알람을 구현한 기본예제이다. 
#
## MainActivity
```
class MainActivity : AppCompatActivity() {

    private lateinit var mBinding:ActivityMainBinding
    private val channelID = "com.khs.notifications.channel1"
    private val KEY_REPLY ="key_reply"
    private var notificationManager:NotificationManager? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        notificationManager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
        createNotificationChannel(channelID,"DemoChannel","This is a demo notification")
        mBinding = DataBindingUtil.setContentView(this,R.layout.activity_main)
        mBinding.apply {
            btnNotify.setOnClickListener {
                displayNotification()
            }
        }
    }

    private fun displayNotification() {
        val notificationId = 45
        // 알림을 클릭했을 경우 SecondActivity
        val intent1 = Intent(this,SecondActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        val tapResultIntent:PendingIntent = PendingIntent.getActivity(
                this, 0, intent1, PendingIntent.FLAG_UPDATE_CURRENT
        )


        // Details를 클릭했을 경우 DetailsActivity
        val intent2 = Intent(this,DetailsActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        val detailsIntent:PendingIntent = PendingIntent.getActivity(
                this, 0, intent2, PendingIntent.FLAG_UPDATE_CURRENT
        )
        val action1:NotificationCompat.Action  = NotificationCompat.Action.Builder(0,"Details",detailsIntent).build()


        // setting 클릭했을 경우 SettingActivity
        val intent3 = Intent(this,SettingActivity::class.java).apply {
            flags = Intent.FLAG_ACTIVITY_CLEAR_TASK
        }
        val settingIntent:PendingIntent = PendingIntent.getActivity(
                this, 0, intent3, PendingIntent.FLAG_UPDATE_CURRENT
        )
        val action2:NotificationCompat.Action  = NotificationCompat.Action.Builder(0,"Setting",tapResultIntent).build()

        //reply action
        val remoteInput: RemoteInput = RemoteInput.Builder(KEY_REPLY).run {
            setLabel("Insert your name here")
            build()
        }
        val replyAction:NotificationCompat.Action = NotificationCompat.Action.Builder(0,"REPLY",tapResultIntent).addRemoteInput(remoteInput).build()


        val notification = NotificationCompat.Builder(this@MainActivity,channelID)
                .setContentTitle("Demo Title")
                .setContentText("This is a demo notification")
                .setSmallIcon(android.R.drawable.ic_dialog_info)
                .setAutoCancel(true)
                .setPriority(NotificationCompat.PRIORITY_HIGH)
                // .setContentIntent(tapResultIntent)
                .addAction(action1)
                .addAction(action2)
                .addAction(replyAction)
                .build()
        notificationManager?.notify(notificationId,notification)
    }

    private fun createNotificationChannel(id:String,name:String,channelDiscription:String){
        if(Build.VERSION.SDK_INT>=Build.VERSION_CODES.O){
            val importance = NotificationManager.IMPORTANCE_DEFAULT
            val channel = NotificationChannel(id,name,importance).apply {
                description =  channelDiscription
            }
            notificationManager?.createNotificationChannel(channel)
        }
    }
}
```
#
## SecondActivity
```
class SecondActivity : AppCompatActivity() {
    private val KEY_REPLY = "key_reply"
    private val channelID = "com.khs.notifications.channel1"
    private val notificationId = 45

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_second)
        receivedInput()
    }

    private fun receivedInput() {
        val intent = this.intent
        val remoteInput = RemoteInput.getResultsFromIntent(intent)
        if (remoteInput != null) {
            val inputString = remoteInput.getCharSequence(KEY_REPLY).toString()
            tv_intent_data.text = inputString
            val replyNotification = NotificationCompat.Builder(this, channelID)
                .setSmallIcon(android.R.drawable.ic_dialog_info)
                .setContentText(inputString)
                .build()
            val manager = getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            manager.notify(notificationId, replyNotification)
        }
    }
}
```