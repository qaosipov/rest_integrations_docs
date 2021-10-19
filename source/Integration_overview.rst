Integration overview
========================

Integration architecture
------------------------

What are the parts of the integration
.....................................
| The broker's integration consists of two independent parts: data integration based on a server-to-server architecture
  and trading integration based on a client-server architecture.

Why data integration is needed
..............................
| The TradingView website can only receive data from the TradingView server. Indicators are counted on this server,
  as well as server alerts, etc. In order for the market data to be first received by the TradingView server, and then
  transferred to the client side, data integration is required.

In what cases it is possible not to integrate data
..................................................
| Market data can come to the TradingView server from another source, for example, directly from the exchange.
  There is no need for market data integration in this case.
  Implement only the `/mapping <https://www.tradingview.com/rest-api-spec/#operation/getMapping>`_
  endpoint to :ref:`map the names<mapping-symbols-label>` of the broker's instruments to TradingView.

How trading integration works
.............................
| The trading integration uses a "client-server" architecture: requests from the user's browser are sent directly to
  the broker's server. The TradingView server is not involved in this data exchange.
  An exception is a request to `/permissions <https://www.tradingview.com/rest-api-spec/#operation/getPermissions>`_.
  It is sent from the TradingView server to give the user access to the data.
  
| Requests from the client browser require a configured :ref:`CORS policy<cors-policy-label>` on the broker side.

General issues
--------------

.. _environments-label:

Different environments
......................

| There are several environment variants are used in the development and support of the integration.
  Each environment has its URL.

- The *production environment* is available to end-users. Real market data is used here. 
  TradingView implements the production environment on its side, the broker on its side.
- The *staging environment* is used for testing. Real market data is not used here. 
  TradingView implements on its side, the broker on its side.
- The *local environment* is used on developers\' computers. Real market data is not used here. 
  Connections to the broker's staging environment are made from ``localhost: 6285``.

| Six options for connecting environments are shown in the table.

.. list-table::
  :widths: auto
  :header-rows: 1

  * - TradingView environment
    - Broker environment
  * - production
    - production
  * - staging
    - production
  * - localhost
    - production
  * - production
    - staging
  * - staging
    - staging
  * - localhost
    - staging

| A TradingView website in a sandbox or production can only be connected to one broker environment at a time.
| You can switch between environments through the browser console.
  Switching instructions are available after configuration by the TradingView team.

.. _cors-policy-label:

Sandbox
.......

What is the Sandbox
''''''''''''''''''''
| The :term:`sandbox` is a fully functional copy of the TradingView website located at ``https://beta-rest.tradingview.com/``.
  Access to the resource is provided by adding an IP address to the whitelist on the TradingView side.

When broker's integration can be placed in the Sandbox
''''''''''''''''''''''''''''''''''''''''''''''''''''''
| There are two conditions to place a broker integration to the sandbox:

- passing conformational tests at `https://www.tradingview.com/rest-api-test/ <https://www.tradingview.com/rest-api-test/>`_
- availability of market data required for the integration to work on the TradingView staging server

| If the broker does not integrate market data but uses data obtained by TradingView from another source,
  it is necessary to implement the `/mapping <https://www.tradingview.com/rest-api-spec/#operation/getMapping>`_ endpoint.

Configuring CORS policy on the broker side
..........................................
| Test servers and website versions in different languages are located on ``*.tradingview.com`` subdomains. 
  For example, the German version of the site is located at ``de.tradingview.com``.
  TradingView can send a request from any of these addresses.

| Therefore, you must include an ``Access-Control-Allow-Origin`` response header 
  with the specific subdomain that sent the request in each endpoint for each response code.

| In addition, in the broker staging environment it is necessary to allow requests from the ``localhost:6285``.
  This address is used on developers\' computers.

Localization support
....................
| Usually, the integration of a specific broker is aimed at an audience using their own national language.
  However, English language support is required for all requests coming from the main locale of the 
  TradingView application.

| The user's locale can be determined through the ``locale`` query parameter, which is present in every request coming 
  from the client to the broker's server.

Adding features after the integration release
................................................
| New features need to be added to the broker's staging environment and tested in the sandbox.
  The feature gets into production only after successful testing by the TradingView testing team.

.. _mapping-symbols-label:

Mapping symbols
---------------

What is mapping
...............
| We call :term:`mapping symbols` the mapping between the names of the broker's instruments and TradingView.
  This mapping avoids the problem of matching TradingView and broker symbol names.

| Mapping is necessary if the broker does not integrate its data in whole or in part on the TradingView servers but
  uses the data already connected.

How to implement mapping
........................
| Mapping is set with the `/mapping <https://www.tradingview.com/rest-api-spec/#operation/getMapping>`_ endpoint 
  implementation. This endpoint must be accessible without authorization.
  In TradingView production, it is automatically requested once a day. Based on the response to the request,
  a mapping of instruments is generated on the TradingView side. 
  In TradingView staging, the ``/mapping`` request is made manually if necessary.
  At the development stage, you can set a partial mapping, i.e. not for all instruments supported by the broker.

How to match symbols
....................
| You can use JSON with a complete list of all symbols to search for a TradingView symbol: 
  ``https://s3.amazonaws.com/tradingview-symbology/symbols.json``. This file is updated daily.

| In response to the ``/mapping`` request, use the ``symbol-fullname`` field value as the TradingView symbol.
  If the broker partially uses TradingView data and partially connects its own, the mapping must be implemented 
  for all symbols.

