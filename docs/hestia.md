```
define HTTP:
    receive(req):
        switch req.folder: # something that means what subfolder of the url it was addressed to
            'flic':
                req -> flic.receive
            'facebook':
                req -> facebook.receive

define credentials:
    init:
        self.creds = {'lifx':     'aaaaaaaa',
                      'facebook': 'bbbbbbbb',
                 

        self.creds['lifx']     ->     lifx.setToken
        self.creds['facebook'] -> facebook.setToken

define watchdog:
    init:
        type resp(status,timestamp)
        self.timeout = 30

        self.status = {'lifx':     resp('no response',timestamp()),
                       'facebook': resp('no response',timestamp())}

        while true:
            true ->     lifx.test
            true -> facebook.test
            wait(5)

    receive(service,status):
        self.status[service] = (status,timestamp())
        temp = {}
        for service,stat in self.status:
            if timestamp() < stat.timestamp + self.timeout:
                temp[service] = stat
            else:
                temp[service] = resp('no response',timestamp())
        self.status = temp
            
    buildReport(ip):
        # blah blah blah
        # html stuff
        # blah blah blah
        # build from self.status
        ip,finishedHTMLreport -> HTTP.send

define flic:
    run(req):
        req.args['button'],req.args['action'] -> main.flic

    test:
        'flic',200 -> watchdog.receive

define main:
    flic(button,action):
        switch button,action:
            'button1','single':
                cmd = {'who':{'lights':'bedroom'},'what':{'state':'on'}}
                cmd,'flic' -> main
            'button1','hold':
                cmd = {'who':{'lights':'bedroom'},'what':{'state':'off'}}
                cmd,'flic' -> main
            'button2','single':
                cmd = {'who':{'lights':'closet'},'what':{'state':'on'}}
                cmd,'flic' -> main
            'button2','hold':
                cmd = {'who':{'lights':'closet'},'what':{'state':'off'}}
                cmd,'flic' -> main
            default:
                'got invalid Flic input',button,action -> logging

    facebook(sender,message):
        # something

    test:
        'main',200 -> watchdog.receive


define facebook:
    init:
        self.url = 'https://graph.facebook.com/v2.6/me/messages?access_token='
        self.accessToken = 'er0fja034fnoaeinrg0a384f0ag'

    receive(req):
        # data = something like req.get_json()
        sender = data['entry'][0]['messaging'][0]['sender']['id']
        message = data['entry'][0]['messaging'][0]['message']['text']

        sender,message -> main.facebook
        
        # gotta do something to return a 200 to FB

    send(userID,message):
        data = {'recipient': {'id': userID},
                'message': {'text': message}}

        {'type':'post','url':url+accessToken,'data':data} -> HTTP.send

    setToken(token):
        self.accessToken = token

    test:
        'facebook',200 -> watchdog.receive

define lifx:
    init:
        headers = {'Authorization': 'Bearer ' + token}

    off(light):
        url = 'https://api.lifx.com/v1/lights/' + light + '/state'
        data = {'power':state}    
        {'type':'put','headers':headers,'url':url,'data':data} -> HTTP.send

    on(light):
        url = 'https://api.lifx.com/v1/lights/' + light + '/state'
        data = {'power':state}    
        {'type':'put','headers':headers,'url':url,'data':data} -> HTTP.send

    toggle(light):
        url = 'https://api.lifx.com/v1/lights/' + light + '/toggle'
        {'type':'post','headers':headers,'url':url,'data':{}} -> HTTP.send

    setToken(token):
        headers = {'Authorization': 'Bearer ' + token}

    test:
        'lifx',200 -> watchdog.receive

define weather:
    init:
        self.lat,self.lon = 0.0,0.0
        
    setLocation(lat,lon):
        self.lat,self.lon = lat,lon



```
