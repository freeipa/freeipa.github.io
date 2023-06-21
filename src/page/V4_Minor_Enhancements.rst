V4_Minor_Enhancements
=====================



Enhancements Addressed in the Current Release
---------------------------------------------

+------------------------+------------------------+-----------------+
| Enhancement            | Comment                | Design document |
+========================+========================+=================+
| https://fedorahosted.o | Add '--nisdomain' and  | N/A             |
| rg/freeipa/ticket/3202 | '--no-nisdomain'       |                 |
|                        | options to the client  |                 |
|                        | installer. By default, |                 |
|                        | NIS domain name is set |                 |
|                        | as the IPA domain      |                 |
|                        | name, unless           |                 |
|                        | '--nisdomain' is       |                 |
|                        | specified, which       |                 |
|                        | overrides the value,   |                 |
|                        | or '--no-nisdomain' is |                 |
|                        | specified, which       |                 |
|                        | disables the           |                 |
|                        | modification of the    |                 |
|                        | NIS domain name.       |                 |
+------------------------+------------------------+-----------------+
| https://fedorahosted.o | ipa-client-install now | N/A             |
| rg/freeipa/ticket/3358 | configures sssd as     |                 |
|                        | sudo provider by       |                 |
|                        | default                |                 |
+------------------------+------------------------+-----------------+
| https://fedorahosted.o | Add ability for        | N/A             |
| rg/freeipa/ticket/4351 | running scripts in     |                 |
|                        | ``ipa console``        |                 |
+------------------------+------------------------+-----------------+
| https://fedorahosted.o | Add the                | N/A             |
| rg/freeipa/ticket/4316 | ``ipa --version``      |                 |
|                        | option                 |                 |
+------------------------+------------------------+-----------------+
| https://fedorahosted.o | Allow unlocking user   | N/A             |
| rg/freeipa/ticket/4407 | in Web UI              |                 |
+------------------------+------------------------+-----------------+
| https://fedorahosted.o | Expose the             | N/A             |
| rg/freeipa/ticket/3306 | krbPrincipalExpiration |                 |
|                        | attribute for editing  |                 |
|                        | in the IPA CLI / WEBUI |                 |
+------------------------+------------------------+-----------------+
|                        |                        |                 |
+------------------------+------------------------+-----------------+



Previous versions
-----------------

See: `V3 Minor Enhancements <V3_Minor_Enhancements>`__