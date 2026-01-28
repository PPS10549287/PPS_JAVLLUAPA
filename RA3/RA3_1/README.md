Introducción 

El presente proyecto documenta el proceso de diseño, implementación y auditoría de una infraestructura web segura basada en el servidor Apache, utilizando una arquitectura de contenedores Docker. El objetivo fundamental es la creación de una Golden Image que consolide diversas capas de protección, siguiendo una metodología incremental de bastionado (hardening). 

A lo largo de las distintas fases, se han integrado medidas de seguridad críticas que abarcan desde la reducción de la superficie de exposición y la gestión de cabeceras HTTP, hasta la implementación de un WAF (Web Application Firewall) con reglas OWASP, la mitigación de ataques de Denegación de Servicio (DoS) y el cifrado de comunicaciones mediante SSL/TLS. Cada etapa hereda las configuraciones de la anterior, garantizando así un sistema robusto, resiliente y alineado con los estándares de seguridad de la industria. 
