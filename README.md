# Linphone4Android
LinPhone是一个网络电话或者IP语音电话（VOIP），是一款遵循GPL的开源的网络视频电话系统，其主要如下：使用linphone，我们可以在互联网上随意的通信，通过语音、视频、即时文本消息。linphone使用SIP协议，是一个标准的开源网络电话系统，你能将linphone与任何基于SIP的VoIP运营商连接起来，包括我们自己开发的免费的基于SIP的Audio/Video服务器。LinPhone是一款自由软件（或者开源软件），你可以随意的下载和在LinPhone的基础上二次开发。LinPhone是可用于桌面电脑：Linux, Windows, MacOSX 以及移动设备：Android, iPhone, Blackberry.

# 接口
可以将项目作为一个Library，进行二次开发，根据业务需求来修改源码。

## 登录
``` java
// 开启线程登录
ServiceWaitThread mThread = new ServiceWaitThread();
mThread.start();

private void syncAccount(String username, String password, String domain) {

    LinphonePreferences mPrefs = LinphonePreferences.instance();
    if (mPrefs.isFirstLaunch()) {
        mPrefs.setAutomaticallyAcceptVideoRequests(true);
//            mPrefs.setInitiateVideoCall(true);
        mPrefs.enableVideo(true);
    }
    int nbAccounts = mPrefs.getAccountCount();
    if (nbAccounts > 0) {
        String nbUsername = mPrefs.getAccountUsername(0);
        if (nbUsername != null && !nbUsername.equals(username)) {
            mPrefs.deleteAccount(0);
            saveNewAccount(username, password, domain);
        }
    } else {
        saveNewAccount(username, password, domain);
        mPrefs.firstLaunchSuccessful();
    }
}

private void saveNewAccount(String username, String password, String domain) {
    LinphonePreferences.AccountBuilder builder = new LinphonePreferences.AccountBuilder(LinphoneManager.getLc())
            .setUsername(username)
            .setDomain(domain)
            .setPassword(password)
            .setDisplayName(Const.LINPHONE_NAME)
            .setTransport(LinphoneAddress.TransportType.LinphoneTransportTcp);

    try {
        builder.saveNewAccount();
    } catch (LinphoneCoreException e) {
        Log.e(e);
    }
}

private class ServiceWaitThread extends Thread {
    public void run() {
        while (!LinphoneService.isReady()) {
            try {
                sleep(30);
            } catch (InterruptedException e) {
                throw new RuntimeException("waiting thread sleep() has been interrupted");
            }
        }
        handler.post(new Runnable() {
            @Override
            public void run() {
                syncAccount(Const.LINPHONE_ACCOUNT, Const.LINPHONE_PWD, Const.HOST);
            }
        });
        mThread = null;
    }
}

```

## 呼出
``` java

private void callOutgoing(String number) {
    try {
        if (!LinphoneManager.getInstance().acceptCallIfIncomingPending()) {
            String to = String.format("sip:%s@%s", number, Const.HOST);
            LinphoneManager.getInstance().newOutgoingCall(to, displayName);

            startActivity(new Intent()
                    .setClass(this, LinphoneActivity.class)
                    .addFlags(Intent.FLAG_ACTIVITY_NEW_TASK));
        }
    } catch (LinphoneCoreException e) {
        LinphoneManager.getInstance().terminateCall();
    }
}

```

**注：该项目只提供登录、呼出两个接口，可以会涉及到修改源码部分来适应，望开发者自己去扩展。**