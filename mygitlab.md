
Установка Gitlab через докер в ручную



version: '3.6'



services:



  web:



    image: 'gitlab/gitlab-ce:latest'



    restart: always



    hostname: 'gitlab.localdomain'



    #    environment:



    #      GITLAB\\\_OMNIBUS\\\_CONFIG: |



    #         external\\\_url 'http://127.0.0.1:8296'



    #        external\\\_url 'http://gitlab.localdomain:8096'



    ports:



      - '8296:80'



      - '443:443'



      - '2222:22'



    volumes:



      - './config:/etc/gitlab'



      - './logs:/var/log/gitlab'



      - './data:/var/opt/gitlab'



    shm\\\_size: '256m'



!\[](images/image20.png)



gitlab очень долго загружается



!\[](images/image14.png)



!\[](images/image22.png)



!\[](images/image8.png)



!\[](images/image43.png)



авторизация не срабатывает, оказалось что в БД создано 0 пользователей



!\[](images/image17.png)



docker exec -it c4593786ea5a gitlab-rake "gitlab:password:reset\\\[root\\]"



gitlab-rails console -e production #если локально



Система сама предложит вам ввести новый пароль и подтвердить его



\\# 1. Находим учетную запись администратора



user = User.find\\\_by\\\_username('root')



\\# 2. Устанавливаем новый пароль (минимум 8 знаков)



user.password = 'НовыйПароль123'



user.password\\\_confirmation = 'НовыйПароль123'



\\# 3. Принудительно разблокируем аккаунт, если он был временно забанен



user.unlock\\\_access! if user.respond\\\_to?(:unlock\\\_access!)



\\# 4. Сохраняем изменения



user.save!



!\[](images/image16.png)!\[](images/image32.png)



!\[](images/image2.png)



!\[](images/image18.png)



В общем все проблема в отсутствии необходимых ресурсов на виртуальной машине



Создадим все заново



!\[](images/image51.png)



192.168.56.10



!\[](images/image4.png)



!\[](images/image47.png)



\[http://gitlab.localdomain/root/my\\\_project.git](https://www.google.com/url?q=http://gitlab.localdomain/root/my\_project.git\&sa=D\&source=editors\&ust=1787296621315150\&usg=AOvVaw3vTgF5AgCTXj0WcLS0jXVj)



 git remote add mygitlab \[http://gitlab.localdomain/root/my\\\_project.git](https://www.google.com/url?q=http://gitlab.localdomain/root/my\_project.git\&sa=D\&source=editors\&ust=1787296621315463\&usg=AOvVaw2DRjBJc\_\_ccgn4bzGSGNfs)



!\[](images/image11.png)



 git push mygitlab



!\[](images/image12.png)



порт(



Попробуем из контейнера



!\[](images/image30.png)В проекте уже что-то есть, создадим проект заново удалив старый



!\[](images/image10.png)



!\[](images/image29.png)



!\[](images/image7.png)



!\[](images/image49.png)



Регистрация раннера:



   docker run -ti --rm --name gitlab-runner \\\\



     --network host \\\\



     -v /srv/gitlab-runner/config:/etc/gitlab-runner \\\\



     -v /var/run/docker.sock:/var/run/docker.sock \\\\



     gitlab/gitlab-runner:latest register



!\[](images/image34.png)



!\[](images/image25.png)



!\[](images/image15.png)



!\[](images/image44.png)



видимо не устраивает порт



!\[](images/image24.png)



http://gitlab.localdomain:8296/



GR1348941JxzJzPevy413zcZ4CLts



runner-01



netology



golang:1.21



!\[](images/image13.png)



!\[](images/image35.png)



volumes = \\\["/cache", "/var/run/docker.sock:/var/run/docker.sock"\\]



!\[](images/image26.png)



Запуск:



   docker run -d --name gitlab-runner --restart always \\\\



     --network host \\\\



     -v /srv/gitlab-runner/config:/etc/gitlab-runner \\\\



     -v /var/run/docker.sock:/var/run/docker.sock \\\\



     gitlab/gitlab-runner:latest



!\[](images/image36.png)



!\[](images/image40.png)



!\[](images/image48.png)



!\[](images/image50.png)



stages:



  - test



  - build



test:



  stage: test



  image: golang:1.17



  script:



   - go test .



build:



  stage: build



  image: docker:latest



  script:



   - docker build .



!\[](images/image37.png)



!\[](images/image5.png)



!\[](images/image33.png)



порт(



git add .gitlab-ci.yml



git remote -v



git commit -am "add CI"



git push mygitlab



подключимся к контейнеру и сделаем файл там



!\[](images/image19.png)



!\[](images/image39.png)



!\[](images/image45.png)



!\[](images/image31.png)



!\[](images/image6.png)



!\[](images/image3.png)



!\[](images/image28.png)



!\[](images/image38.png)



!\[](images/image21.png)



порт(



Если прописать



 environment:



      GITLAB\\\_OMNIBUS\\\_CONFIG: |



        external\\\_url 'http://gitlab.localdomain:8296'



то сайт перестает открываться.



гугл:



Проблема кроется в параметре external\\\_url 'http://gitlab.localdomain:8296' внутри GITLAB\\\_OMNIBUS\\\_CONFIG.Когда вы указываете там порт :8296, встроенный в GitLab веб-сервер (Nginx) начинает слушать входящие соединения именно на порту 8296 внутри контейнера. Но в секции ports у вас написано '8296:80', то есть Docker пытается перенаправить трафик с внешнего порта 8296 на 80-й порт контейнера, который теперь пуст. В итоге соединение сбрасывается.



либо '8296:8296



либо



environment:



      GITLAB\\\_OMNIBUS\\\_CONFIG: |



        external\\\_url 'http://gitlab.localdomain:8296'



        nginx\\\['listen\\\_port'\\] = 80 # Принудительно заставляем Nginx слушать порт 80



version: '3.6'



services:



  web:



    image: 'gitlab/gitlab-ce:latest'



    restart: always



    hostname: 'gitlab.localdomain'



    environment:



      GITLAB\\\_OMNIBUS\\\_CONFIG: |



        external\\\_url 'http://gitlab.localdomain:8296'



        nginx\\\['listen\\\_port'\\] = 80



    ports:



      - '8296:80'



      - '443:443'



      - '2222:22'



    volumes:



      - './config:/etc/gitlab'



      - './logs:/var/log/gitlab'



      - './data:/var/opt/gitlab'



    shm\\\_size: '256m'



!\[](images/image23.png)



Gitlab запустился. но ранер работать не хочет



!\[](images/image42.png)



docker run --add-host=gitlab.localdomain:192.168.56.10 — чтобы сам раннер видел GitLab



Попробуем добавить в фаил с настройками



sudo nano /srv/gitlab-runner/config/config.toml



!\[](images/image9.png)



!\[](images/image41.png)



!\[](images/image46.png)



Sonarqube



vagrant ssh



cd /vagrant



docker-compose up -d



sudo sysctl -w vm.max\\\_map\\\_count=524288



!\[](images/image27.png)



!\[](images/image1.png)



система не справляется с контейнерами, сонакуб не хочет запускаться

