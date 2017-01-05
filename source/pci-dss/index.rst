.. _pci_dss:

Guide for PCI DSS
=================

.. rubric:: Introduction

The **Payment Card Industry Data Security Standard (PCI DSS)** is a proprietary information security standard for organizations that handle branded credit cards from the major card schemes including *Visa*, *MasterCard*, *American Express*, *Discover*, and *JCB*. The standard was created to increase controls around cardholder data to reduce credit card fraud.

Wazuh helps to implement PCI DSS by performing log analysis, file integrity checking, policy monitoring, intrusion detection, real-time alerting and active response. This guide explains how these capabilities help with each of the standard requirements:

* `Wazuh for PCI DSS Guide (PDF) <http://ossec.wazuh.com/ruleset/PCI_Guide.pdf>`_
* `Wazuh for PCI DSS Guide (Excel) <http://ossec.wazuh.com/ruleset/PCI_Guide.xlsx>`_

In the following section we will elaborate on some specific use cases. They explain how to use Wazuh capabilities to meet the standard requirements.

.. toctree::
    :maxdepth: 1
    :caption: Contents

    log-analysis
    policy-monitoring
    rootkit-detection
    file-integrity-monitoring
    active-response
    elastic

What's next
-----------

Once you know how Wazuh can help with PCI DSS, we encourage you to move forward and try out Elastic integration or the Wazuh ruleset, check them out at:

* :ref:`Elastic Stack integration guide <installation_elastic>`
* :ref:`Wazuh Ruleset <ruleset>`
