.. _openscap_module:


OpenScap
========

The **OpenSCAP wodle** is an integration of `OpenScap <https://www.open-scap.org/>`_ in *Wazuh HIDS* that provides the ability to perform configuration and vulnerability scans of an agent. Mainly it allows:

 - **Security compliance**: It is a state where computer systems are in line with a specific *security policy* or a *security benchmark*. These policies define security requirements which all systems of an organization must comply with.

 - **Vulnerability assessment**: It is a process that identifies and classifies vulnerabilities on a system.

 - **Specialized assessment**: It is a process that performs a specific set of checks. For example, a policy to detect suspicious file names and suspicious location of files.

.. topic:: Documentation sections

   .. toctree::
      :maxdepth: 1

      oscap-settings
      oscap-examples
      oscap-faq


Brief introduction to SCAP
--------------------------

 The `Security Content Automation Protocol (SCAP) <https://scap.nist.gov/>`_ is a specification for expressing and manipulating security data in standardized ways. SCAP uses several individual specifications in concert, in order to automate continuous monitoring, vulnerability management, and security policy compliance evaluation reporting.

 Process of security compliance evaluation:

  - **SCAP scanner**: It is an application that reads a SCAP policy and checks whether or not the system is compliant with it. There are many `tools <https://nvd.nist.gov/scapproducts.cfm>`_ to scan your systems. This wodle is an integration with the NIST-certified scanner: **OpenSCAP**.

  - **Security policies (SCAP content)**: They determine how a system must be set up and what to check for. These policies contain machine-readable descriptions of the rules which your system will be required to follow.

   - **Profiles**: Each security policy can contain multiple profiles, which provide sets of rules and values implemented according to a specific security baseline. You can think of a profile as a particular subset of rules within the policy; the profile determines which rules defined in the policy are selected (checked) and what values are used during the evaluation.

  - **Evaluation (scan)**: It is the process to evaluate a policy with a SCAP scanner. The process usually takes a few minutes, depending on the number of selected rules.

Requirements
^^^^^^^^^^^^

This wodle is executed on the agent, so each one must meet the following requirements:

Wazuh HIDS
  Wodles are part of *OSSEC Wazuh fork*, so install it following these :ref:`Installation guide <installation>`.


OpenScap
  In order to perform SCAP evaluations we need the scanner. As we mentioned above, we use OpenSCAP. You can install it on RedHat or CentOS versions 6 and 7 with this command: ::

    yum install openscap-scanner


Python 2.6+
  Python is a core part of this wodle. Currently all Linux distributions come with python, so it should not be an inconvenience.
