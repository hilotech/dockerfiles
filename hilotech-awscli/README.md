AWS-CLI
=======

CentOS 6.5��AWS-CLI���l�ߍ��킹��Dockerfile�ł��B


�g�p���@
--------

* Dockerfile�̐擪�ɂ���ϐ���K���ɐݒ肵�Ă�������
    * _SMTP_RELAY     ���[�����M�̂��߂Ƀ����[���Ă��炤SMTP�T�[�o�[
        * �����[����SMTP�T�[�o�[�iDocker�z�X�g�����j�ɂ́APostfix�Ȃ�  
          /etc/postfix/main.cf ��  
          mynetworks = 172.0.0.0/8, 127.0.0.0  
          smtpd_recipient_restrictions = permit_mynetworks, reject  
          ��ǉ����Ă�������
* Dockerfile��u�����f�B���N�g����  
  # docker build --no-cache --rm -t hilotech/awscli ./  
  �ȂǂƂ��ăC���[�W���r���h
* �N����  
  # docker run -d hilotech/awscli  
  �Ȃ�
* SSH��iam���[�U�[��password�p�X���[�h�ŃA�N�Z�X�ł���悤�ɂȂ���
  ����̂Łi���{��ɂȂ��ĂȂ��c�j�Adocker ps �Ń^�[�Q�b�g�̃R���e�i
  ID���`�F�b�N���Ă���A  
  # ssh iam@`docker inspect --format="{{ .NetworkSettings.IPAddress }}" �R���e�iID`  
  �Ƃ�����ƃ��O�C���ł��܂�

