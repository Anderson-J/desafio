FROM nginx:latest

COPY opencms /etc/nginx/sites-enabled/opencms

COPY nginx.conf /etc/nginx/nginx.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]