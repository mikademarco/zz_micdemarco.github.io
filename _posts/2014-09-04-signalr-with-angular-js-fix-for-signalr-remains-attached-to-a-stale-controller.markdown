---
author: micdemarco
comments: true
date: 2014-09-04 14:03:09+00:00
layout: post
link: https://micdemarco.wordpress.com/2014/09/04/signalr-with-angular-js-fix-for-signalr-remains-attached-to-a-stale-controller/
slug: signalr-with-angular-js-fix-for-signalr-remains-attached-to-a-stale-controller
title: SignalR with Angular JS - Fix for SignalR remains attached to a stale controller
wordpress_id: 5
tags:
- angular
- firebootcamp
- hub
- model
- signalr
---

While working with AngularJS and SignalR on a project at firebootcamp.com we ran into a situation where our SignalR hub was not updating our page when we navigated away and then navigated back to the page using angular routing.

The following video describes the problem:

https://www.youtube.com/watch?v=7khwTLkfv5Q

We had a SignalR hub defined and started within our controller:

    
    vm.hub = $.connection.hub;
    vm.scanHub = $.connection.scanHub;
    vm.hub.start().done()
    vm.scans = [];


We had our SignalR client function updating the vm.scans array inside our controller:

    
    vm.scanHub.client.updateScan = function (data) {
        var i = 0;
        for (i = 0; i < vm.scans.length; i++) {
            if (vm.scans[i].id == data.id) {
                vm.scans[i].status = data.status;
                vm.scans[i].status = data.status;
                vm.scans[i].goodLinks = data.goodLinks;
                vm.scans[i].badLinks = data.badLinks;
                vm.scans[i].totalLinks = data.totalLinks;
                return;
            }
        }
    };


This function worked fine on the first load, but when we navigated away from the page, we realised that although the hub was still active and the function was being called, it was not updating our view.

While debugging we found that the hub was remaining attached to a stale controller that was out of scope.  SignalR was not restarting the hub for the new instance of the controller that was created when we navigated back to our page.

In order to fix this we found that if we stopped the hub whenever we changed route, then the hub started up again correctly when we navigated back to the initial page.

The following code fixed our issue:

    
    app.run([
        '$rootScope', function ($rootScope, security) {
             
            $rootScope.$on('$routeChangeStart', function (event, currRoute, prevRoute) {
                $.connection.hub.stop();
            });
    }]);


Conclusion:

If you are going to use SignalR within a controller, remember to stop the hub before starting it again in the controller scope.  The new controller will not start a new instance of the hub if the previous page controller still has an active hub attached to it.

Alternatively use a service to inject your SignalR hub as described in this excellent blog post:

http://sravi-kiran.blogspot.com.au/2013/09/ABetterWayOfUsingAspNetSignalRWithAngularJs.html

Thanks [@benjii22](https://twitter.com/benjii22) [@igor_goldobin](https://twitter.com/igor_goldobin) [@cameronchong](https://twitter.com/cameronchong) for helping to find the solution.