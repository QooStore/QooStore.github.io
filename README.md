# QooStore.github.io

## Exercise JQuary

(function(window, $) {
    'use strict';

    var runningFunction, updateFunction, installFunction, agentStatus, agenetcheckTm, currentAgentStatus, agentOptions, version_info, agentCheckCount = 0;
    var isGetAgentVersion = false;
    var isAgentUpdating = false;
    var isAgentInstalled = false;
    var isAgentInstalling = false;
    var agentCheckTimeoutCount = 20;
    var agentCheckTimeoutCountUnderIE8 = 7;
    var isUnderIE8 = false;
    var _timeoutCount = 5000;
    var isUsePingCheck = false;

    var AGENT_STATUS_READY = 'READY';
    var AGENT_STATUS_NOT_RUNNING = 'NOT_RUNNING';
    var AGENT_STATUS_INSTALL = 'INSTALL';
    var AGENT_STATUS_UPDATE = 'UPDATE';
    var AGENT_STATUS_RUNNING = 'RUNNING';
    var AGENT_STATUS_CHECKING = 'CHECKING';
    var AGENT_STATUS_TIMEOUT = 'TIMEOUT';

    var test_display = null;
    var testFontWidth = 0;
    var testFontHeight = 0;
    var installedFontWidth = 0;
    var installedFontHeight = 0;
    var guid = 0;
    var isOpenedNewWindow = false;

    function _registerEvent(target, eventType, cb) {
        if (target.addEventListener) {
            target.addEventListener(eventType, cb);
            return {
                remove: function() {
                    target.removeEventListener(eventType, cb);
                }
            };
        } else {
            target.attachEvent(eventType, cb);
            return {
                remove: function() {
                    target.detachEvent(eventType, cb);
                }
            };
        }
    }

    function _createHiddenIframe(target, uri) {
        var iframe = document.createElement('iframe');
        iframe.src = uri;
        iframe.id = 'hiddenIframe';
        iframe.style.display = 'none';
        target.appendChild(iframe);
        return iframe;
    }

    function openUriWithHiddenFrame(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri With HiddenFrame', true, false);
        /////////////////////////////////////////////////////

        var timeout = setTimeout(function() {
            failCb();
            handler.remove();
        }, _timeoutCount);

        var iframe = document.querySelector('#hiddenIframe');
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, 'about:blank');
        }

        var handler = _registerEvent(window, 'blur', onBlur);

        function onBlur(event) {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }

        iframe.contentWindow.location.href = uri;
    }

    function openUriWithTimeoutHack(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri With Timeout Hack', true, false);
        /////////////////////////////////////////////////////

        var timeout = setTimeout(function() {
            failCb();
            handler.remove();
        }, _timeoutCount);

        var handler = _registerEvent(window, 'blur', onBlur);

        function onBlur() {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }

        window.location = uri;
    }

    function openUriUsingFirefox(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri With Firefox', true, false);
        /////////////////////////////////////////////////////
        var iframe = document.querySelector('#hiddenIframe');
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, 'about:blank');
        }
        try {
            iframe.contentWindow.location.href = uri;
            successCb();
        } catch (e) {
            if (e.name == 'NS_ERROR_UNKNOWN_PROTOCOL') {
                failCb();
            }
        }
    }

    function openUriUsingIE(uri, failCb, successCb) {
        //check if OS is Win 8 or 8.1
        var ua = navigator.userAgent.toLowerCase();

        var isWin8 = /windows nt 6.2/.test(ua) || /windows nt 6.3/.test(ua);
        var isEdge = /Edge\/12./i.test(ua);
        var ieVer = getInternetExplorerVersion();

        if (isWin8) {
            //openUriUsingIEInWindows8(uri, failCb, successCb);
            openUriWithHiddenFrame(uri, failCb, successCb);
        } else if (isEdge) {
            openUriUsingEdgeInWindows10(uri, failCb, successCb);
        } else {
            if (ieVer === 10) {
                openUriUsingIE10InWindows7(uri, failCb, successCb);
            } else if (ieVer === 9 || ieVer === 11) {
                openUriWithHiddenFrame(uri, failCb, successCb);
            } else {
                openUriInNewWindowHack(uri, failCb, successCb);
            }
        }
    }

    function openUriUsingEdgeInWindows10(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri Using Edge In Windows 10', true, false);
        /////////////////////////////////////////////////////

        var timeout = setTimeout(function() {
            failCb();
            handler.remove();
        }, _timeoutCount);

        var iframe = document.querySelector('#hiddenIframe');
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, 'about:blank');
        }

        var handler = _registerEvent(iframe, 'blur', onBlur);

        function onBlur(event) {
            clearTimeout(timeout);
            handler.remove();
            successCb();
        }

        iframe.src = uri;
    }

    function openUriUsingIE10InWindows7(uri, failCb, successCb) {
        var timeout = setTimeout(failCb, _timeoutCount);
        window.addEventListener('blur', function() {
            clearTimeout(timeout);
            successCb();
        });

        var iframe = document.querySelector('#hiddenIframe');
        if (!iframe) {
            iframe = _createHiddenIframe(document.body, 'about:blank');
        }
        try {
            iframe.contentWindow.location.href = uri;
        } catch (e) {
            failCb();
            clearTimeout(timeout);
        }
    }

    function openUriInNewWindowHack(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri Using New Window Hack', true, false);
        /////////////////////////////////////////////////////

        var myWindow = window.open('', '', 'width=0,height=0');

        myWindow.document.write('<iframe src="' + uri + '"></iframe>');
        setTimeout(function () {
            try {
                myWindow.location.href;
                myWindow.setTimeout('window.close()', 1000);
                successCb();
            } catch (e) {
                myWindow.close();
                failCb();
            }
        }, 1000);

        /*if(isOpenedNewWindow==false) {
            isOpenedNewWindow=true;
            var myWindow = window.open('', '', 'width=0,height=0');

            //myWindow.document.write('<iframe src='' + uri + ''></iframe>');
            setTimeout(function () {
                try {
                    myWindow.location.href = uri;
                    myWindow.setTimeout('window.close()', 1000);
                    successCb();
                } catch (e) {
                    myWindow.close();
                    failCb();
                }
            }, 1000);
        }*/

        // var myWindow = window.open('', '', 'width=0,height=0');
        //
        // //myWindow.document.write('<iframe src='' + uri + ''></iframe>');
        // setTimeout(function() {
        //     try {
        //         //myWindow.document.domain="kollus.com";
        //         if (url.indexOf('http') > -1) {
        //             myWindow.document.domain = window.location.host;
        //         }
        //         console.log(myWindow.document.domain);
        //         myWindow.location.href = uri;
        //         myWindow.setTimeout('window.close()', 1000);
        //         successCb();
        //     } catch (e) {
        //         myWindow.close();
        //         failCb();
        //     }
        // }, 1000);
    }

    function openUriUsingIEInWindows8(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('- open Uri Using IE In Windows 8', true, false);
        /////////////////////////////////////////////////////

        if (navigator.msLaunchUri) {
            navigator.msLaunchUri(uri,
                function() {
                    window.location = uri;
                    successCb();
                },
                failCb
            );
        }
    }

    function checkBrowser() {
        var ua = navigator.userAgent.toLowerCase();

        var isOpera = !!window.opera || navigator.userAgent.indexOf(' OPR/') >= 0;
        var isEdge = /Edge\/12./i.test(ua);
        var browser = {
            isOpera: isOpera,
            isFirefox: typeof InstallTrigger !== 'undefined',
            isSafari: Object.prototype.toString.call(window.HTMLElement).indexOf('Constructor') > 0,
            isChrome: !!window.chrome && !isOpera && !isEdge,
            isIE: /*@cc_on!@*/false || !!document.documentMode,   // At least IE6
            isEdge: isEdge
        };

        var returnValue = '';
        for (var key in browser) {
            if (browser[key] === true) {
                //Logger/////////////////////////////////////////////
                window.logger('- Check browser : ' + key, true, false);
                /////////////////////////////////////////////////////
            }
        }
        return browser;
    }

    function getInternetExplorerVersion() {
        var rv = -1, ua, re;
        if (navigator.appName === 'Microsoft Internet Explorer') {
            ua = navigator.userAgent;
            re = new RegExp('MSIE ([0-9]{1,}[\.0-9]{0,})');
            if (re.exec(ua) !== null)
                rv = parseFloat(RegExp.$1);
        }
        else if (navigator.appName === 'Netscape') {
            ua = navigator.userAgent;
            re = new RegExp('Trident/.*rv:([0-9]{1,}[\.0-9]{0,})');
            if (re.exec(ua) !== null) {
                rv = parseFloat(RegExp.$1);
            }
        }

        //Logger/////////////////////////////////////////////
        window.logger('- InternetExplorer Version : ' + rv);
        /////////////////////////////////////////////////////

        return rv;
    }

    function protocolCheck(uri, failCb, successCb) {
        //Logger/////////////////////////////////////////////
        window.logger('Agentcheck Initialize...', true, false);
        /////////////////////////////////////////////////////
        var browser = checkBrowser();

        function failCallback() {
            return failCb && failCb();
        }

        function successCallback() {
            return successCb && successCb();
        }

        if (browser.isFirefox || browser.isOpera) {
            openUriUsingFirefox(uri, failCallback, successCallback);
        } else if (browser.isChrome) {
            openUriWithTimeoutHack(uri, failCallback, successCallback);
        } else if (browser.isIE || browser.isEdge) {
            openUriUsingIE(uri, failCallback, successCallback);
        } else {
            //not supported, implement please
        }
    }

    window.Agentcheck = function() {
        //Logger////////////////////////////////
        window.logger('Agentcheck initialize start...', true, false);
        ////////////////////////////////////////
    };

    Agentcheck.prototype.options = function(obj) {
        $.each(obj, function(key, value) {
            //Logger/////////////////////////////////////////////
            window.logger('- options : {' + key + ': ' + value + '}', true, false);
            /////////////////////////////////////////////////////
        });

        agentOptions = obj;

        if ($('#testFont').length) return;

        $('head').append('<' + 'style id="testFontStyle"> #testFont {visibility:hidden; position:absolute; font-size: 50px!important; font-family: "";}</' + 'style>');
        $('body').append($('<div id="testFontDisplay" />'));
        $('#testFontDisplay').append('<span id="testFont" class="fonttest">' + agentOptions.test_content + '</span>');

        testFontWidth = $('#testFontDisplay').find('span').width();
        testFontHeight = $('#testFontDisplay').find('span').height();
    };

    Agentcheck.prototype.start = function() {
        if (agentOptions.agentCheckTimeoutInterval !== undefined) {
            agentCheckTimeoutCount = agentOptions.agentCheckTimeoutInterval;
        }

        var ie_ver = getInternetExplorerVersion() * 1;
        // 브라우저가 IE8 이하 버전인 경우 FontSize 비교로직을 태우지 않는다.
        if (checkBrowser().isIE == true && ie_ver <= 8) {
            isUnderIE8 = true;
            getAgentVersion();
        } else {
            guid++;

            var style = '<' + 'style id="installedFontStyle"> #installedFont' + guid + ' {visibility:hidden; position:absolute; font-size: 50px!important; font-family: "' + agentOptions.detect_font_name + '"; } <' + '/style>';

            $('head').find('#installedFontStyle').remove().end().append(style);
            $('body').append($('<div id="installedFontDisplay" />'));
            $('#installedFontDisplay').append('<span id="installedFont' + guid + '" class="fonttest">' + agentOptions.test_content + '</span>');

            installedFontWidth = $('#installedFontDisplay').find('span').width();
            installedFontHeight = $('#installedFontDisplay').find('span').height();

            $('#testFontWidth').text('(width : ' + testFontWidth + ', height : ' + testFontHeight + ')');
            $('#installedFontWidth').text('(width : ' + installedFontWidth + ', height : ' + installedFontHeight + ')');

            isAgentInstalled = ((testFontWidth != installedFontWidth) || (testFontHeight !== installedFontHeight));
            //Logger/////////////////////////////////////////////
            window.logger('Kollus Player V3 Agent is installed : ' + isAgentInstalled, true, false);
            /////////////////////////////////////////////////////

            if (typeof agentOptions.timeoutCount !== 'undefined') {
                _timeoutCount = agentOptions.timeoutCount;
            }

            if (isAgentInstalled) {
                getAgentVersion();
            } else {
                setAgentStatus(AGENT_STATUS_INSTALL);
            }
        }
        setAgentcheckInterval();
        removeTestText();
    };

    Agentcheck.prototype.getVersion = function() {
        return version_info;
    };

    function removeTestText() {
        $('head').find('#installedFontStyle').remove();
        $('head').find('#testFontStyle').remove();
        $('body').find('#installedFontDisplay').remove();
        $('body').find('#testFontDisplay').remove();
    }

    /*
    * @status agent status string
    * - AGENT_STATUS_READY = 'READY';
    * - AGENT_STATUS_NOT_RUNNING = 'NOT_RUNNING';
    * - AGENT_STATUS_INSTALL = 'INSTALL';
    * - AGENT_STATUS_UPDATE = 'UPDATE';
    * - AGENT_STATUS_RUNNING = 'RUNNING';
    * - AGENT_STATUS_CHECKING = 'CHECKING';
    * - AGENT_STATUS_TIMEOUT = 'TIMEOUT';
    */
    function setAgentStatus(status) {
        agentStatus = status;
    }

    function getAgentStatus() {
        return (typeof agentStatus !== 'undefined') ? agentStatus : AGENT_STATUS_READY;
    }

    function setAgentcheckInterval() {
        agenetcheckTm = setInterval(function() {
            var status = getAgentStatus();
            if ((status == AGENT_STATUS_CHECKING && isGetAgentVersion === false) || currentAgentStatus !== status) {
                currentAgentStatus = status;

                //Logger/////////////////////////////////////////////
                window.logger('=========================================', true, false);
                window.logger('# Agent status change : ' + currentAgentStatus, true, false);
                window.logger('=========================================', true, false);
                /////////////////////////////////////////////////////
                switch (currentAgentStatus) {
                    case AGENT_STATUS_NOT_RUNNING:
                        agentExecute();
                        break;

                    case AGENT_STATUS_INSTALL:
                        getAgentVersion(false, true);
                        break;

                    case AGENT_STATUS_UPDATE:
                        agentOptions.updateFn();
                        agentExecute(true);
                        break;

                    case AGENT_STATUS_RUNNING:
                        agentOptions.runningFn();
                        clearInterval(agenetcheckTm);
                        break;

                    case AGENT_STATUS_CHECKING:
                        getAgentVersion();
                        break;

                    case AGENT_STATUS_TIMEOUT:
                        agentOptions.agentCheckTimeoutFn();
                        clearInterval(agenetcheckTm);
                        break;
                }
            }

        }, 250);
    }

    function getAgentVersion(isUpdateCheck, isInstallCheck) {
        isGetAgentVersion = true;

        if (typeof isUpdateCheck == 'undefined') {
            isUpdateCheck = false;
        }
        if (typeof isInstallCheck == 'undefined') {
            isInstallCheck = false;
        }

        // var checkTimeoutValue = (Math.pow(agentCheckCount, 2) * 100);
        // if (checkTimeoutValue < 1000) {
        //     checkTimeoutValue = 1000;
        // }
        var checkTimeoutValue = 1000;

        //Logger/////////////////////////////////////////////
        window.logger('Get agent version : ' + agentOptions.agentURL + '/version?path=' + agentOptions.versionCheckURL, true, false);
        window.logger('- isUpdateCheck : ' + isUpdateCheck + ', - isInstallCheck : ' + isInstallCheck, true, false);
        /////////////////////////////////////////////////////

        var ajax_url = agentOptions.agentURL + '/version?path=' + agentOptions.versionCheckURL;
        var ajax_opt = {
            crossDomain: true,
            async: true,
            cache: false,
            timeout: 10000,
            dataType: 'json',
            contentType: 'text/plain',
            success: function(result, status, jqxhr) {
                //Logger/////////////////////////////////////////////
                window.logger('Agent is running, is Update : ' + result.info.update, true, false);
                /////////////////////////////////////////////////////

                isGetAgentVersion = false;

                var isAgentUpdate = result.info.update;

                if (isUpdateCheck) {
                    setAgentStatus(AGENT_STATUS_CHECKING);
                } else {
                    if (isAgentUpdate) {
                        if (isAgentUpdating === false) {
                            isAgentUpdating = true;
                            setAgentStatus(AGENT_STATUS_UPDATE);
                        }
                    } else {
                        version_info = result;
                        setAgentStatus(AGENT_STATUS_RUNNING);
                    }
                }
            },
            error: function(jqxhr, status, errorThrown) {
                //Logger/////////////////////////////////////////////
                window.logger('Agent is not running...', true, false);
                /////////////////////////////////////////////////////

                if (isUnderIE8) {
                    if (agentCheckCount < agentCheckTimeoutCountUnderIE8) {
                        agentCheckCount++;

                        //Logger/////////////////////////////////////////////
                        window.logger('AgentCheck Timeout Count (UnderIE8) : ' + agentCheckCount, true, false);
                        /////////////////////////////////////////////////////

                        isGetAgentVersion = false;

                        if (getAgentStatus() == AGENT_STATUS_READY) {
                            setAgentStatus(AGENT_STATUS_NOT_RUNNING);
                        } else {
                            setAgentStatus(AGENT_STATUS_CHECKING);
                        }
                    } else {
                        agentCheckCount = 0;
                        isUnderIE8 = false;
                        setAgentStatus(AGENT_STATUS_INSTALL);
                    }
                } else {
                    if (agentCheckCount < agentCheckTimeoutCount) {
                        agentCheckCount++;

                        //Logger/////////////////////////////////////////////
                        window.logger('AgentCheck Timeout Count : ' + agentCheckCount, true, false);
                        /////////////////////////////////////////////////////

                        isGetAgentVersion = false;

                        if (getAgentStatus() == AGENT_STATUS_READY) {
                            setAgentStatus(AGENT_STATUS_NOT_RUNNING);
                        } else {
                            if (isInstallCheck) {
                                agentOptions.installFn();
                            }
                            setAgentStatus(AGENT_STATUS_CHECKING);
                        }
                    } else {
                        setAgentStatus(AGENT_STATUS_TIMEOUT);
                    }
                }
            }
        };

        setTimeout(function() {
            //console.log('###check time : ' + checkTimeoutValue);
            $.ajax(ajax_url, ajax_opt);
        }, checkTimeoutValue);
    }

    function agentExecute(isUpdate) {
        var exeURL = (isUpdate === true) ? agentOptions.updateURL : agentOptions.executeURL;
        //var exeURL = agentOptions.updateURL;

        if (isUsePingCheck) {
            var is_https = window.location.href.indexOf("https://") == 0;
            var proxy_url = (is_https) ? "https://proxy.catoms.net:18389" : "http://127.0.0.1:18388";

            //Logger/////////////////////////////////////////////
            window.logger('Agent Ping check : ' + proxy_url, true, false);
            /////////////////////////////////////////////////////

            var ajax_url = proxy_url + '/ping';
            var ajax_opt = {
                crossDomain: true,
                async: false,
                cache: false,
                timeout: 1000,
                complete: function (jqxhr, status) {
                    if (jqxhr.status == 200) {
                        exeURL = proxy_url + "/" + exeURL;
                    }
                    agentExecuteWithURL(isUpdate, exeURL);
                }
            }
            $.ajax(ajax_url, ajax_opt);
        } else {
            agentExecuteWithURL(isUpdate, exeURL);
        }
    }

    function agentExecuteWithURL(isUpdate, exeURL) {
        //Logger/////////////////////////////////////////////
        window.logger('Agent Execute with URL : ' + exeURL + ', isUpdate : ' + isUpdate, true, false);
        /////////////////////////////////////////////////////

        var is_custom_scheme = exeURL.indexOf('kollus://') == 0;

        if (is_custom_scheme) {
            protocolCheck(exeURL,
                function() { //fail
                    //Logger/////////////////////////////////////////////
                    window.logger('- Protocol check fail', true, false);
                    /////////////////////////////////////////////////////

                    setAgentStatus(AGENT_STATUS_CHECKING);
                },
                function() { //success
                    //Logger/////////////////////////////////////////////
                    window.logger('- Protocol check success', true, false);
                    /////////////////////////////////////////////////////

                    getAgentVersion(true);
                }
            );
        } else {
            var ajax_opt = {
                crossDomain: true,
                async: false,
                cache: false,
                timeout: 1000,
                success: function (result, status, jqxhr) {
                    if (jqxhr.status == 200) {
                        //Logger/////////////////////////////////////////////
                        window.logger('- Protocol check success', true, false);
                        /////////////////////////////////////////////////////

                        getAgentVersion(true);
                    }
                },
                error: function (jqxhr, status, errorThrown) {
                    //Logger/////////////////////////////////////////////
                    window.logger('- Protocol check fail', true, false);
                    /////////////////////////////////////////////////////

                    setAgentStatus(AGENT_STATUS_CHECKING);
                }
            }
            $.ajax(exeURL, ajax_opt);
        }
    }
})(window, jQuery);